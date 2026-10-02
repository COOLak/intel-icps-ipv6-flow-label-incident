# Prior reports: Intel was told, and closed the thread

This defect isn't new. Users have publicly reported the same failure pattern on Intel's connectivity software since at least 2021. In **November 2025**, an engineer gave Intel the exact mechanism on Intel's own forum. Intel sent a questionnaire, then closed the thread.

## The November 2025 report on Intel Community

**[Intel Connectivity Network Service is causing the ipv6 flowtable attribute set to 0](https://community.intel.com/t5/Wireless/Intel-Connectivity-Network-Service-is-causing-the-ipv6-flowtable/m-p/1725616)** (Intel Community, Wireless board)

- The author identifies Intel Connectivity Performance Suite, delivered through **Windows Update** as "Intel Corporation - SoftwareComponent - 4.1025.207.1". When the TLS handshake starts, it clears the IPv6 flow label. The author explains that this breaks ECMP in large cloud networks, because the flow is "randomly send to different backend device after tcp handshake". The post includes a `curl -6` reproduction against an Azure endpoint, and stopping the service fixes it.
- Intel moved the thread to another forum and asked for details. When there was no answer, it closed the inquiry: *"As I haven't received a response, I will proceed to close this inquiry."*
- Later replies from other organizations report the same symptom: **M365 Search and SharePoint fail with `ERR_CONNECTION_RESET`** on IPv6, and disabling the Intel service fixes it. One adds that removing the software *"took just one Windows Update and the problem was back"*.

The author re-posted when the first thread was lost: [second thread, November 2025](https://community.intel.com/t5/Wireless/Intel-Connectivity-Network-Service-is-causing-the-ipv6-flowtable/m-p/1727313).

## Earlier reports of the same pattern (Killer / Intel connectivity software)

| Date | Where | What users saw | What fixed it |
|---|---|---|---|
| 2021–2022 | [jQuery CDN issue on GitHub](https://github.com/jquery/codeorigin.jquery.com/issues/80) | `curl -6` reset, `curl -4` fine. The CDN found nothing on its side. | Owners of Dell XPS and Alienware machines traced it to Killer software |
| 2022 | [Microsoft Q&A](https://learn.microsoft.com/en-us/answers/questions/4340112/random-connection-reset-error) | Connection resets only on Microsoft sites (Bing, Office, SharePoint, LinkedIn) | Uninstalling or disabling Killer, or disabling IPv6. Windows Update reinstalled Killer. |
| 2022–2025 | [Intel Community](https://community.intel.com/t5/Wireless/Killer-Network-Service-preventing-connection-to-some-websites/td-p/1366144) | Resets only over IPv6 | Disabling the Killer Network Service. Intel blamed the user's ISP. |
| 2023–2024 | [Intel Community](https://community.intel.com/t5/Wireless/How-to-permanently-disable-Killer-Prioritization-Engine/m-p/1469486) | Teams/SharePoint, OneDrive sign-in and Outlook broken. One user spent three weeks with Microsoft 365 support, then paid for a technician. | Disabling the Killer Prioritization Engine |
| 2024 | [Microsoft Q&A](https://learn.microsoft.com/en-us/answers/questions/3739394/err-connection-reset-on-some-sites) | `ERR_CONNECTION_RESET` on some sites, including Bing | Disabling the Killer services or IPv6 |
| 2025 | [Brave Community](https://community.brave.app/t/err-connection-reset-errors-on-new-win-11-laptops/637663) | Resets on several new Lenovo laptops | Disabling the Intel connectivity service |
| 2026 | [Vivaldi forum](https://forum.vivaldi.net/topic/114868/websites-show-of-error-err_connection_reset?page=2) | `ERR_CONNECTION_RESET` | Stopping the Intel Connectivity Network Service |

A public remediation script for IT administrators also warns that Intel Connectivity Performance Suite ["may cause network issues when IPv6 is enabled"](https://github.com/realmjoin/realmjoin-remediation/blob/main/intel/remove-intel-connectivity-performance-suite/README.md).

The reports above are summarized from the linked public pages. The November 2025 Intel thread was checked directly on 2026-10-02.

## What this case adds

- **Wire-level proof** at the Wi-Fi miniport: SYN 24/24 labels kept, ACK/data 0/42. The RST comes from a different machine (hop limit 49 vs 47).
- **The exact component**: the `IpV6 Egress Transport Callout` in `IntcCo11X64.sys` 11.5.11.19, registered with `FWP_CALLOUT_FLAG_CONDITIONAL_ON_FLOW`.
- **A controlled A/B test** showing that stopping only the bandwidth-management service doesn't help, while stopping the traffic-control driver fixes 100% of connections.
- **The consumer impact**: OneNote desktop silently showing empty notebooks as "Up to date" for nearly three weeks, plus an escalated Microsoft support case that never found the cause.

## Why it matters

Intel had the mechanism in writing eleven months before this report and closed the thread. Users have been told to blame their ISP, reinstall Office or pay for technicians. Windows Update keeps putting the software back after people remove it.
