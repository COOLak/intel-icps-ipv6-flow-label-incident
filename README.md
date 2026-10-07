# Intel Connectivity Performance Suite breaks IPv6, OneNote and Microsoft 365

> **Status, October 7, 2026:** Reproduced again on the OEM build, and **Intel's current generic ICPS 50.26.623.243 does not have the bug** (same laptop, same day: 138/170 IPv6 connections with the OEM build, 169/170 with the generic one). The broken OEM build 40.25.926.173 reached the laptop through **Windows Update**. No advisory, and no fix in the OEM channel yet. Affected users can disable ICPS or replace it with the generic build: see the [workaround](workaround.md).

This repository is a public, privacy-sanitized evidence hub for a defect in **Intel Connectivity Performance Suite (ICPS)**, the "network optimizer" that ships preinstalled on Intel laptops. Its kernel driver rewrites the **IPv6 Flow Label** to zero partway through every TCP connection. Microsoft's edge network uses that label to route packets. So the rest of each connection lands on a server that never saw it start, and that server resets it.

From the user's side this looks like random **"connection reset"** errors on many websites, and a **OneNote that can't reach its notebooks** while claiming they are "Up to date". IPv4 is unaffected, which is why the web and phone versions kept working and nobody could explain the failure.

> **Intel was told in September 2025.** An engineer posted this exact mechanism on Intel Community ("Intel Connectivity Network Service is causing the ipv6 flowtable attribute set to 0"), from an HP laptop. Intel closed the inquiry, the engineer re-posted in November, and Intel closed it again. Similar reports go back to 2021. See [prior reports](prior-reports.md).

> **Not only Samsung.** Microsoft's Update Catalog lists the builds reported as broken under seven submitters: Samsung, Lenovo, VAIO, ASUS, HP, Dynabook and Huaqin. Of 1,086 ICPS entries from 19 companies, only one is the build shown fixed, and it couldn't reach this laptop. HP, ASUS and Lenovo owners report the same IPv6 resets. See [Not only Samsung](affected-oems.md).

## Start here

- **[Open the public incident page](https://coolak.github.io/intel-icps-ipv6-flow-label-incident/)**
- **[Not only Samsung: who ships the affected builds](affected-oems.md)**
- **[Prior reports: Intel was told, and closed the thread twice](prior-reports.md)**
- **[Am I affected? Check and fix in five minutes](workaround.md)**
- **[Full technical analysis: wire evidence and A/B proof](technical-analysis.md)**
- **[What Intel, Microsoft and Samsung need to do](owner-action.md)**
- **[Where it has been reported, and the status of each report](vendor-reports.md)**
- **[Dated timeline](timeline.md)**
- **[Machine-readable state](incident-state.json)**

## Short summary

| | |
|---|---|
| **Component** | Intel Connectivity Performance Suite 40.25.926.0. Driver `IntcCo11X64.sys` 11.5.11.19, "Intel Connectivity Traffic Control Callout Driver" (service `INTCCoSvc`), WFP provider "Rivet Networks, LLC - RFE version 6.1.0.1" |
| **Shipped on** | Preinstalled by the OEM. Observed on a Samsung Galaxy Book (960QHA, Intel Core Ultra 7 256V, Intel Wi-Fi 7 BE201) |
| **OS** | Windows 11 Pro 26H2, build 26300.9457 |
| **Defect** | The SYN leaves with the stack's IPv6 Flow Label. Every ACK and data segment after it leaves with Flow Label `0`. On the wire, **0 of 42** post-handshake packets kept their label. |
| **Why it breaks things** | RFC 6437 expects the label to stay constant for the life of a flow. Microsoft's Azure Front Door hashes on it, so the label-0 packets go to a different backend, which answers with RST. |
| **Impact** | 60–100% of new IPv6 connections to Microsoft's edge reset within one round trip. OneNote desktop sync dead for nearly three weeks. "Connection reset" on other sites. |
| **Proof** | With ICPS running: 4/10 connections OK, 0/42 labels kept. With the ICPS driver stopped: 10/10 OK, 65/65 labels kept. Notebooks synced within seconds. |
| **October 7 retest** | OEM 40.25.926.0 re-enabled: 138/170 IPv6 connections OK across 34 sites, 1,385 packets with the label zeroed; OneDrive, OneNote sync, Office, Teams, Skype, Azure portal, Azure DevOps and Visual Studio Marketplace resetting. Intel generic 50.26.623.243 (driver 12.10.14.33): 169/170 OK, 11,711/11,711 labels kept. |
| **Distribution** | Windows Update installed the OEM ICPS packages on the affected laptop on 2025-05-06 (4.1025.304.2), 2026-05-09, 2026-05-14 and 2026-06-13 (40.25.926.173). |
| **Workaround** | Disable `INTCCoSvc`, `IDBWM`, `Intel Connectivity Network Service` and `IntelConnectService`, or replace the OEM build with Intel's generic 50.26.623.243. See [workaround](workaround.md). |

## Who is responsible for what

- **Intel** shipped a kernel driver that breaks a basic IPv6 rule, on every connection, invisibly. Its current generic build no longer does, but the fix was silent: no advisory, nothing in the release notes, and no visible effort to get OEMs off the broken builds. Users in Intel's forum were told to uninstall the software, and the broken builds keep reaching laptops through OEM packages on Windows Update. A fix that the affected users never receive doesn't fix anything for them.
- **Samsung** preinstalled it and published the broken 40.25.926.173 build on Windows Update. Its one fixed submission targets different device IDs and never reached this laptop. The owner never chose the software and had no way to know it was there. Lenovo, VAIO, ASUS, HP and Dynabook publish broken builds too.
- **Microsoft** delivered the broken build through Windows Update, three times in 2026 on the affected laptop, and its hardware program signed it. Its edge falls over on the mangled packets, where Google, Cloudflare and Akamai do not. OneNote reports the dead connection as "Up to date". An escalated Microsoft support case opened in August never found the cause and has gone quiet.

The details and specific asks are in [owner-action.md](owner-action.md).

## How it was found

Ordinary troubleshooting couldn't find this. Sign-out, cache resets, Office repair, reinstalling and "use the web version" are all irrelevant to it.

In this case, an escalated Microsoft support case ran for five weeks without finding the cause, and OpenAI's Codex gave up on it. The cause was then found in a few hours of packet-level work by Anthropic's **Claude**, run as an agent on the affected machine, without knowing about Intel's earlier forum thread. Claude timed the resets against the round trip, captured traffic at the Wi-Fi miniport and decoded the IPv6 headers. It then inventoried every Windows Filtering Platform callout and switched network filter drivers off one at a time, reversibly and with elevation, until only the Intel driver was left.

## Privacy

Private correspondence, account identifiers, support ticket numbers and the machine's IP addresses are summarized or omitted. The Microsoft endpoint addresses quoted are public anycast addresses.
