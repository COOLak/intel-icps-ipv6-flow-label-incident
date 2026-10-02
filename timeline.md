# Timeline

All times are UTC.

## 2021–2025: the same pattern, reported again and again

Users of Killer and Intel connectivity software publicly report IPv6-only connection resets, often only to Microsoft services. They are told to check their ISP or reinstall Office, and are sent from one vendor to the next. See [prior reports](prior-reports.md).

## 2025-11: Intel is given the exact mechanism

A post on Intel Community explains that the Intel connectivity service clears the IPv6 flow label at the TLS handshake and breaks cloud ECMP load balancing. Intel asks for details, then closes the inquiry for lack of a response. In 2026, other organizations add Microsoft 365 and SharePoint `ERR_CONNECTION_RESET` reports to the same thread. One notes that Windows Update reinstalled the software.

## 2026-08-27: first reports to Microsoft

OneNote desktop shows notebooks as empty while reporting "Up to date". The pattern is documented: Microsoft endpoints reset over IPv6 and work over IPv4. It is reported through Microsoft Support article feedback, a public [Feedback Portal idea](https://feedbackportal.microsoft.com/feedback/idea/dd326ebc-9da1-f111-85ce-7c1e529382f4) and a Microsoft Support chat. The chat case is escalated for product review. The cause is not identified.

## 2026-09-12, 20:59 UTC: OneNote sync fails again

OneNote's own logs show all three notebooks entering an error state ("We can't get to your notebooks right now"). They stay broken for nearly three weeks.

## 2026-10-02: Codex gives up

An OpenAI Codex session investigates the sync failure and abandons it without finding a cause.

## 2026-10-02: Claude finds the cause

Anthropic's Claude, running as an agent on the affected machine, takes over:

- It confirms that IPv6 connections to Microsoft's Azure Front Door edge are reset within one round trip, while IPv4 is clean.
- It captures traffic at the Wi-Fi miniport. Every post-handshake packet leaves with IPv6 Flow Label 0, and the RST comes from a different machine than the SYN-ACK (hop limit 49 vs 47).
- It inventories the Windows Filtering Platform callouts and isolates the AdGuard, AdGuard VPN, NetLimiter and Intel filter drivers one at a time.

## 2026-10-02, 15:21 UTC: fix confirmed

With the Intel Connectivity Performance Suite driver (`INTCCoSvc`) and its services stopped, 10/10 IPv6 connections succeed and 65/65 packets keep their flow label. ICPS is left disabled: 10/10 connections, 73/73 labels. All three OneNote notebooks sync within seconds. Microsoft 365 sign-in works in Chromium browsers again.

## 2026-10-02: reported to all three vendors

- **Microsoft:** full root cause sent into the existing support case. Two public Feedback Hub reports filed ([OneNote](https://aka.ms/AA13q37q), [Windows networking](https://aka.ms/AA13q37r)). Comment added to the Feedback Portal idea. Answer posted on [Microsoft Q&A](https://learn.microsoft.com/en-us/answers/questions/5682454/onenote-sync-says-up-to-date-on-old-pc-but-noteboo).
- **Intel, 16:47 UTC:** Intel Customer Support case opened and acknowledged.
- **Intel, 17:08 UTC:** [Intel Community post](https://community.intel.com/t5/Wireless/Intel-ICPS-driver-zeroes-IPv6-Flow-Label-mid-connection-breaking/m-p/1760647) published on the Wireless board.
- **Samsung:** Samsung Community US report posted, then hidden by Samsung's spam filter.
- This public tracker is created.
