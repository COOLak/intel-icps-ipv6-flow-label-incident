# Timeline

All times are UTC.

## 2021–2025: the same pattern, reported again and again

Users of Killer and Intel connectivity software publicly report IPv6-only connection resets, often only to Microsoft services. They are told to check their ISP or reinstall Office, and are sent from one vendor to the next. See [prior reports](prior-reports.md).

## 2025-09 and 2025-11: Intel is given the exact mechanism, twice

A post on Intel Community, from an HP ZBook owner, explains that the Intel connectivity service clears the IPv6 flow label at the TLS handshake and breaks cloud ECMP load balancing. Intel asks for details, then closes the inquiry for lack of a response, on 2025-09-22. The author re-posts in November, and Intel closes it again on 2025-11-12. In June 2026, organizations running HP EliteBooks add Microsoft 365 and SharePoint `ERR_CONNECTION_RESET` reports to the same thread. One notes that Windows Update reinstalled the software.

## 2026-08-27: first reports to Microsoft

OneNote desktop shows notebooks as empty while reporting "Up to date". The pattern is documented: Microsoft endpoints reset over IPv6 and work over IPv4. It is reported through Microsoft Support article feedback, a public [Feedback Portal idea](https://feedbackportal.microsoft.com/feedback/idea/dd326ebc-9da1-f111-85ce-7c1e529382f4) and a Microsoft Support chat. The chat case is escalated for product review. The cause is not identified.

## 2025-05 to 2026-06: Windows Update delivers ICPS

Windows' driver setup log on the affected laptop shows the Windows Update client installing ICPS packages: 4.1025.304.2 on 2025-05-06, then the 40.25.926.173 extension on 2026-05-09, the 40.25.926.173 component on 2026-05-14, and the extension again on 2026-06-13.

## 2026-07: another Intel Community report

A user with Intel Wi-Fi 7 BE201 and ICPS 40.25.725.165 reports intermittent IPv6 failures against AWS CloudFront. Intel answers that it has no confirmed public advisory for the behavior. See [prior reports](prior-reports.md).

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
- **Samsung:** Samsung Community US report posted, then hidden by Samsung's spam filter. Samsung Care support case opened through live chat, with escalation to Galaxy Book engineering requested.
- This public tracker is created.

## 2026-10-06: Intel asks for a test with its generic package

Intel Customer Support says the OEM package may differ from Intel's generic releases, and asks for a clean removal followed by a test with the generic ICPS 50.26.623.243.

## 2026-10-07: reproduced, and the generic build does not have the bug

- **10:28–10:33 UTC:** the original failure is reproduced on the OEM build. Re-enabling ICPS 40.25.926.173 drops IPv6 connections from 168/170 to 138/170 across 34 websites, with OneDrive, OneNote sync, Office, Teams, Skype, the Azure portal, Azure DevOps and the Visual Studio Marketplace resetting. Disabling it again restores 169/170.
- **11:58–12:07 UTC:** the OEM packages are removed using the method in Intel's support article 000093451, the Store app and leftovers are removed with Revo Uninstaller, and the PC is restarted.
- **12:34 UTC:** with no ICPS installed, 168/170 connections succeed and every packet keeps its flow label.
- **12:39 UTC:** with Intel's generic ICPS 50.26.623.243 (driver 12.10.14.33) installed and its filters active, 169/170 succeed and all 11,711 post-handshake packets keep their flow label. See [technical analysis](technical-analysis.md#october-7-reproduced-again-and-intels-generic-build-does-not-have-the-bug).
- The results are sent to Intel. The Microsoft support case is asked to escalate beyond OneNote, after six weeks without an update.

## 2026-10-07: Samsung support has no escalation path

- **16:03–17:05 UTC, Samsung US live chat:** across four agents, Samsung says the October 2 case was "only for documentation", that it has no software engineering escalation department, and that the owner should contact Amazon or call a US phone line. A supervisor offers basic troubleshooting or a repair, neither of which can change a driver package delivered through Windows Update, then gives the address of Samsung Electronics America's Office of the President.
- A formal complaint is sent to the Office of the President, asking for the fixed package on Windows Update and a written reply from a named contact within 14 days.
- An [update](https://us.community.samsung.com/t5/Computers/Galaxy-Book-ships-Intel-software-that-breaks-IPv6-networking-and/m-p/3683502/highlight/true#M14838) is posted on the Samsung Community thread.
- Intel is sent the list of other affected laptop makers. See [affected laptop makers](affected-oems.md).
- Intel Customer Support replies (by 2026-10-08) that it has passed requests to its team for an advisory naming the affected and first fixed ICPS versions, notice to every OEM shipping affected builds, work with Microsoft to stop Windows Update offering them, and an engineering escalation on which driver version fixed the flow label. Intel promises a status update by October 13.
- A letter is sent to Microsoft's Office of the CEO, asking for a named owner for the support case.

## 2026-10-08: Microsoft confirms the mechanism and starts work on a fix

- A relationship manager from Microsoft Executive Customer Relations opens a new case, which it says was generated by a communication sent to Microsoft's executives, and asks for a phone contact. Contact details are sent the same day.
- Microsoft's Azure Networking engineering leadership confirms the problem: when a client changes the IPv6 flow label after the SYN, routers that hash on the label deliver the rest of the connection to a different Azure Front Door server. Microsoft says it has asked its team to work with the Windows team on a fix, and is working on the Azure Front Door side to reduce the impact. No date is given.
- A senior Microsoft leader passes the support case to his escalation team.

## 2026-10-09: Samsung opens an escalation; call with Microsoft

- **14:23–14:32 UTC, Samsung US live chat:** a supervisor opens an escalation ticket for the Office of the President under Samsung Electronics America, and says the team will make contact "within 24 to 48 business hours". No contact has come as of 2026-10-16.
- The relationship manager from Microsoft Executive Customer Relations calls the owner. The full write-up is sent the same day.
- The complaint sent to Samsung's US Office of the President, about a US-model Galaxy Book bought in the US, is forwarded to Samsung Russia, apparently because the owner's email address ends in .ru (the owner does not live in Russia). On 2026-10-09 Samsung Russia's customer service replies that it "do[es] not have the opportunity to help you on this issue and transfer the information to the Galaxy Book Software development department", and recommends contacting "the representative office of the country where the laptop was purchased": Samsung in the US, where the complaint started. A reply to the Office of the President the same day points this out. Two days earlier, Samsung's US live chat had told the owner: "we do not have a specific software engineering escalation department". No answer as of 2026-10-16.

## 2026-10-13: Intel refers the OEM build to Samsung

- Intel Customer Support replies that the ICPS package on the laptop is Samsung's OEM build, recommends asking Samsung for its latest OEM version, and says that the advisory, OEM notification and Microsoft coordination requests fall within Samsung's scope as the OEM. It acknowledges in writing, citing the owner's tests, that its generic ICPS 50.26.623.243 (driver 12.10.14.33) "does not exhibit the IPv6 Flow Label zeroing behavior".
- Intel is asked, as the maker of the driver, whether driver 11.5.11.19 in the OEM package is Intel's own unmodified build, which driver version first keeps the flow label, whether Intel has told the laptop makers, and what came of each request it listed on October 8. Microsoft's Update Catalog lists the same 40.25.926.173 build under Lenovo, VAIO and Huaqin as well as Samsung. A written answer is requested by October 21.

## 2026-10-14: Intel says it can provide nothing further

- **00:58 UTC:** Intel Customer Support replies to the questions sent on October 13 that "The information previously shared represents the full extent of what we are currently able to provide". On the request for a contact at a higher level, it says: "I am actively coordinating this on my end. You will be reached out to as soon as possible." No such contact has come as of 2026-10-16.

## 2026-10-14: Microsoft Executive Customer Relations names no engineering owner or date

- **04:48 UTC:** Microsoft Executive Customer Relations replies. It turns down the goodwill gesture the owner had asked for and calls its decision final. It thanks the owner for the effort put into "documenting and isolating this behavior" and acknowledges "how difficult this issue would have been to identify from the symptoms alone", but names no review channel, reference, owner or date for a Windows Update fix.
- Replies the same day ask for a review of the decision by an Executive Customer Relations manager, with the manager's name and written reasoning; the reference and owning team of the engineering review it referred to; the date when Windows Update will stop offering or replace builds 40.25.926.173, 40.25.725.165 and 4.1025.207.1 (or a written statement that Microsoft will not give a date); and how to deliver the logs. A written answer is requested by October 21.

## 2026-10-16: Microsoft Executive Customer Relations points to Feedback Hub, with no engineering owner or date

- **03:57 UTC:** Microsoft Executive Customer Relations replies again. It gives no engineering reference, owning team or date for Windows Update to stop offering or replace the three builds reported broken. It says its decision "remains separate from any technical review of the reported driver behavior" and that the technical information "may be reviewed through the appropriate product-feedback channel". For the logs, it sends a ten-step guide to Feedback Hub, calls Feedback Hub "the appropriate Windows channel for reporting the driver and Windows Update behavior", and adds: "A submission does not guarantee an individual response, an engineering owner, or a remediation date."
- On the goodwill gesture, which the owner had asked for without citing any contract, license or warranty, it cites the warranty and liability limits of the Windows license, writes that "Microsoft did not commission or enter into an agreement to purchase your investigation, testing, or findings", and says its decision "will not be reconsidered through this case". The reply is signed by the same relationship manager and names no manager to review the decision.
- The owner's two public Feedback Hub reports, [OneNote](https://aka.ms/AA13q37q) and [Windows networking](https://aka.ms/AA13q37r), each with the full root-cause report attached, were filed on 2026-10-02. The owner had told Executive Customer Relations about them on 2026-10-14, and had said that Feedback Hub does not answer the request for a reference, owning team and date.
- No further message from Microsoft Support has been received in the support case since it was escalated in late August: no reply to the root cause sent on 2026-10-02, and none to the request of 2026-10-07 for a case owner.
- Microsoft's Azure Networking engineering, which confirmed the mechanism in writing on 2026-10-08, the day after receiving the evidence, has given no date for a fix and has not answered a request for a timeline.
- **19:49–19:52 UTC, Microsoft Update Catalog:** the three builds reported broken are still listed, with the same entry counts as on 2026-10-07: 40.25.926.173 (23 entries), 40.25.725.165 (39) and 4.1025.207.1 (8). The build shown fixed, 50.26.623.243, still has a single entry, for Windows 11 25H2 and later only. See [affected laptop makers](affected-oems.md).
- No Intel advisory has been published.

## 2026-10-20: Intel is "still actively reviewing"; the Catalog is unchanged

- **02:09 UTC:** a different Intel Customer Support agent writes that Intel is "still actively reviewing this matter internally". The reply answers none of the questions sent on 2026-10-13. It asks whether Samsung has sent any update, and says the case will be "temporarily closed" if there is no reply by 2026-10-22. The higher-level contact promised on 2026-10-14 has not been in touch.
- **05:40 UTC, Microsoft Update Catalog:** the three builds reported broken are still listed with the same entry counts: 40.25.926.173 (23), 40.25.725.165 (39) and 4.1025.207.1 (8). The fixed 50.26.623.243 still has a single entry.
- **05:49 UTC:** a reply asks Intel to keep the case open. It reports that Samsung has sent nothing since its 2026-10-09 escalation ticket. It also gives Microsoft's status: the mechanism is confirmed and work is under way on the Azure Front Door side and with the Windows team, but there is no date, and the Catalog is unchanged. It repeats the questions only Intel can answer: whether driver 11.5.11.19 is Intel's unmodified build, which driver version first keeps the flow label, whether Intel has told the laptop makers, and the status of each action Intel listed on 2026-10-08.
- No contact from Samsung since 2026-10-09, and no Microsoft date. No Intel advisory has been published.
