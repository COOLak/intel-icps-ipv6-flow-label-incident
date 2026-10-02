# Intel Connectivity Performance Suite breaks IPv6, OneNote and Microsoft 365

> **Status, October 2, 2026:** Root cause confirmed with packet captures and a controlled A/B test. Reported to Intel, Microsoft and Samsung. **No vendor fix yet.** Affected users can apply the [workaround](workaround.md) today.

This repository is a public, privacy-sanitized evidence hub for a defect in **Intel Connectivity Performance Suite (ICPS)**, the "network optimizer" that ships preinstalled on Intel laptops. Its kernel driver rewrites the **IPv6 Flow Label** to zero partway through every TCP connection. Microsoft's edge network uses that label to route packets. So the rest of each connection lands on a server that never saw it start, and that server resets it.

From the user's side this looks like random **"connection reset"** errors on many websites, and a **OneNote that can't reach its notebooks** while claiming they are "Up to date". IPv4 is unaffected, which is why the web and phone versions kept working and nobody could explain the failure.

> **Intel was told in November 2025.** An engineer posted this exact mechanism on Intel Community ("Intel Connectivity Network Service is causing the ipv6 flowtable attribute set to 0"). Intel closed the inquiry for lack of a response. Similar reports go back to 2021. See [prior reports](prior-reports.md).

## Start here

- **[Open the public incident page](https://coolak.github.io/intel-icps-ipv6-flow-label-incident/)**
- **[Prior reports: Intel was told, and closed the thread](prior-reports.md)**
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
| **Workaround** | Disable `INTCCoSvc`, `IDBWM`, `Intel Connectivity Network Service` and `IntelConnectService`. See [workaround](workaround.md). |

## Who is responsible for what

- **Intel** ships a kernel driver that breaks a basic IPv6 rule, on every connection, invisibly.
- **Samsung** preinstalled it on the laptop. The owner never chose it and had no way to know it was there.
- **Microsoft** runs an edge that falls over on it, where Google, Cloudflare and Akamai do not. Microsoft also ships a OneNote that reports a dead connection as "Up to date". An escalated Microsoft support case opened in August never found the cause.

The details and specific asks are in [owner-action.md](owner-action.md).

## How it was found

Ordinary troubleshooting couldn't find this. Sign-out, cache resets, Office repair, reinstalling and "use the web version" are all irrelevant to it.

In this case, an escalated Microsoft support case ran for five weeks without finding the cause, and OpenAI's Codex gave up on it. The cause was then found in a few hours of packet-level work by Anthropic's **Claude**, run as an agent on the affected machine, without knowing about Intel's earlier forum thread. Claude timed the resets against the round trip, captured traffic at the Wi-Fi miniport and decoded the IPv6 headers. It then inventoried every Windows Filtering Platform callout and switched network filter drivers off one at a time, reversibly and with elevation, until only the Intel driver was left.

## Privacy

Private correspondence, account identifiers, support ticket numbers and the machine's IP addresses are summarized or omitted. The Microsoft endpoint addresses quoted are public anycast addresses.
