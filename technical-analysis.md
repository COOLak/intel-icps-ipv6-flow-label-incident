# Technical analysis

Intel Connectivity Performance Suite zeroes the IPv6 Flow Label mid-connection, and Microsoft's edge resets the connection.

## Environment

- Samsung Galaxy Book, model 960QHA (BIOS P17ALY.390.260616.03). Intel Core Ultra 7 256V (Lunar Lake). Intel Wi-Fi 7 BE201, driver 23.160.0.4.
- Windows 11 Pro 26H2, build 26300.9457. OneNote (Microsoft 365) 16.0.20430.20092.
- **Intel Connectivity Performance Suite** 40.25.926.0, preinstalled by the OEM:
  - `IntcCo11X64.sys` 11.5.11.19, "Intel Connectivity Traffic Control Callout Driver", service `INTCCoSvc`
  - WFP provider "Rivet Networks, LLC - RFE version 6.1.0.1"
  - Services `IDBWM` (Intel Dynamic Bandwidth Management) 1.19.0.0, `Intel Connectivity Network Service` 40.25.926.173, `IntelConnectService`
- Native IPv6 from the ISP.
- Microsoft endpoint: Azure Front Door anycast `2620:1ec:50::11` / `150.171.22.11` (`ln-0001.ln-msedge.net`, the nearest Microsoft edge), serving `d.docs.live.net`, `docs.live.net` and `api.onedrive.com`.

## Symptoms

- OneNote for Windows: "We can't get to your notebooks right now". Logs show `SoapCallFailed 0x803D0014` (`WS_E_ENDPOINT_DISCONNECTED`) and WinHTTP error `12030`. Notebooks opened as **empty shells while showing "Up to date"**. OneNote's own logs show all three notebooks stuck in error from 2026-09-12 20:59 UTC.
- Chromium browsers: `ERR_CONNECTION_RESET` on Microsoft 365 sign-in (`m365.cloud.microsoft`) and on other sites.
- OneNote on the web and on a phone kept working. They reached the same service over IPv4, or over a different network.

## What happens on the wire

Packets were captured with `pktmon` at the Wi-Fi miniport, below every filter driver, and the IPv6 headers were decoded.

1. The Windows TCP/IP stack sends the **SYN** with a non-zero IPv6 Flow Label, for example `0xf196f`.
2. Every **ACK and data segment** after the handshake leaves with **Flow Label 0**:

   | Packets leaving the PC | Kept their flow label |
   |---|---|
   | SYN | 24/24 |
   | FIN | 11/11 |
   | ACK/data | **0/42** |

   Later runs gave the same picture: 0/42 and 0/46.
3. Microsoft's edge load-balances IPv6 on the flow label (RFC 6438 / RFC 7098 style hashing):
   - The SYN reaches backend A, which answers with a SYN-ACK at hop limit **47**.
   - The label-0 ACK hashes to a different node, which answers with a **RST** at hop limit **49**. A different hop limit means a different machine sent it.
   - Backend A never sees the ACK. It retransmits its SYN-ACK at +0.4 s and +1.2 s, then sends its own RST.
4. Result: **60–100% of new IPv6 TCP connections** to the Microsoft edge are reset within one round trip (about 5 ms), **even when the client sends zero bytes**.
5. IPv4 has no flow label and was **0% affected**. Google, Cloudflare and Akamai over IPv6 were also unaffected, presumably because they don't hash on the label.

RFC 6437 says the flow label should stay constant for the life of a flow. A component that rewrites it partway through a TCP connection does not conform.

## Which component does it

The ICPS callout **`IpV6 Egress Transport Callout`** is registered at `FWPM_LAYER_OUTBOUND_TRANSPORT_V6` with **`FWP_CALLOUT_FLAG_CONDITIONAL_ON_FLOW`**. That flag means it only acts once a flow exists. That fits exactly: the SYN goes out untouched, and everything after it is rewritten. A second callout, `IpV6 Ingress Network Outbound Callout`, sits at `FWPM_LAYER_OUTBOUND_IPPACKET_V6`. Packets that are absorbed and re-injected (or cloned) come out with the label set to 0.

## Controlled A/B test

Each step was elevated and reversible, and was measured with 10 IPv6 connections to the Microsoft edge plus a packet capture.

| Condition | IPv6 connections OK | ACK/data keeping flow label |
|---|---|---|
| Baseline, everything running | 4/10 | **0/42 (0%)** |
| `IDBWM` (Dynamic Bandwidth Management) stopped | 1/10 | 0/18 (0%) |
| **`INTCCoSvc` driver and ICPS services stopped** | **10/10** | **65/65 (100%)** |
| Final state, ICPS kept disabled | **10/10** | **73/73 (100%)** |

Stopping the bandwidth-management service alone did not help. Stopping the traffic-control driver fixed it completely.

**Ruled out.** Each of these was disabled on its own, and the label stripping and resets continued every time: the AdGuard WFP driver, the AdGuard VPN WFP driver and the NetLimiter 5 driver. Microsoft Defender Firewall, NDU and the other Windows WFP callouts stayed active throughout. None of them strips the label once ICPS is gone.

## Outcome

With `INTCCoSvc`, `IDBWM`, `Intel Connectivity Network Service` and `IntelConnectService` disabled:

- All three OneNote notebooks synced immediately.
- Microsoft 365 sign-in loads in Chromium browsers.
- 10/10 IPv6 connections succeed, and every packet keeps its flow label.

## October 7: reproduced again, and Intel's generic build does not have the bug

Intel Customer Support asked for a test with Intel's own generic ICPS instead of the OEM package. Before that, the original result was reproduced once more on the same laptop. Each step below made 5 IPv6 connections to each of 34 websites (170 in total), with header-only `pktmon` captures at the Wi-Fi miniport and an IPv4 control.

| Step | ICPS | Traffic-control driver | IPv6 connections OK | Post-handshake packets keeping their flow label | To the OneDrive/OneNote edge (`2620:1ec:50::11`) |
|---|---|---|---|---|---|
| A | OEM 40.25.926.0 installed, disabled | stopped | 168/170 | 3779/3779 | all kept |
| B | **OEM 40.25.926.0 re-enabled** | `IntcCo11X64.sys` **11.5.11.19** | **138/170** | **8457/9842** (1385 set to 0) | **0/60** |
| C | OEM disabled again | stopped | 169/170 | 4240/4240 | all kept |
| N | OEM removed (Intel article 000093451 method), reboot | none | 168/170 | 4230/4230 | all kept |
| G | **Intel generic 50.26.623.243** installed and running | `IntcCo11X64.sys` **12.10.14.33** | **169/170** | **11711/11711** | **201/201** |

The single miss in steps A, C, N and G is `www.microsoft.com` (Akamai) timing out, which also happens over IPv4 and is unrelated.

**With the OEM build running (step B), these failed over IPv6 and worked over IPv4 at the same moment:**

| Service | Host | IPv6 OK in B | In A, C, N and G |
|---|---|---|---|
| OneNote / OneDrive sync | `d.docs.live.net` | 1/5 | 5/5 |
| OneDrive | `api.onedrive.com` | 1/5 | 5/5 |
| Microsoft 365 / Office | `www.office.com` | 3/5 | 5/5 |
| Microsoft Teams | `teams.microsoft.com` | 4/5 | 5/5 |
| Skype | `www.skype.com` | 2/5 | 5/5 |
| Azure portal | `portal.azure.com` | **0/5** | 5/5 |
| Azure DevOps | `dev.azure.com` | 2/5 | 5/5 |
| Visual Studio Marketplace | `marketplace.visualstudio.com` | **0/5** | 5/5 |

In step B the OEM driver zeroed the label on traffic to **every** destination, Google, Cloudflare and Wikipedia included. Only services whose load balancers hash on the label actually reset the connections.

**The generic build hooks the same places.** In step G, twelve Rivet/RFE filters were active, including `Rivet IpV6 Outbound Transport Filtering Layer` → `IpV6 Egress Transport Callout`. In step N there were none. The newer driver processes the same packets without zeroing the label. Intel's release notes for 50.26.623.243 and 50.26.220.212 don't mention a fix.

**Windows Update delivered the broken build.** Windows' own driver setup log (`setupapi.dev.log`) records every ICPS install on this laptop as an "Install Windows Update driver" operation run by the Windows Update client (`wuaucltcore.exe`):

| Date (UTC) | Package | Version |
|---|---|---|
| 2025-05-06 | `ICPSExtension.inf` and `ICPSComponent.inf` | 4.1025.304.2 |
| 2026-05-09 | `ICPSExtension.inf` | **40.25.926.173** |
| 2026-05-14 | `ICPSComponent.inf` | **40.25.926.173** |
| 2026-06-13 | `ICPSExtension.inf`, again | **40.25.926.173** |

After the generic 50.26.623.243 was installed, a Windows Update driver scan on 2026-10-07 offered no driver updates, so Windows Update did not try to replace the newer build with the older one.

## Method

The investigation was run by Anthropic's Claude as an agent on the affected machine, after OpenAI's Codex had abandoned the same problem. It used:

- per-source-address socket experiments,
- timing of the RST against the round-trip time,
- NIC-level packet captures with IPv6 header decoding,
- an inventory of every Windows Filtering Platform provider and callout,
- elevated, reversible isolation of each filter driver in turn.
