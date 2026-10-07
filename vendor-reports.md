# Vendor reports and status

Last updated: **2026-10-07**. "Filed" means submitted or published. It does not mean a vendor has acknowledged, reproduced or fixed anything.

## Intel

| Channel | Visibility | Status |
|---|---|---|
| [Intel Community, Wireless board](https://community.intel.com/t5/Wireless/Intel-ICPS-driver-zeroes-IPv6-Flow-Label-mid-connection-breaking/m-p/1760647) | Public | Posted 2026-10-02. |
| Intel Customer Support case | Private | Opened 2026-10-02. On 2026-10-06 Intel asked for a test with its generic ICPS package. Results sent 2026-10-07: the generic 50.26.623.243 does not have the bug. Asked Intel which version fixed it, to get the fix to OEMs and Windows Update, and to publish an advisory. On 2026-10-07, sent Intel the list of other affected laptop makers and organizations. Awaiting answer. |

## Microsoft

| Channel | Visibility | Status |
|---|---|---|
| [Feedback Hub: OneNote](https://aka.ms/AA13q37q) | Public | Filed 2026-10-02 with the full root-cause report attached. |
| [Feedback Hub: Windows networking / WFP](https://aka.ms/AA13q37r) | Public | Filed 2026-10-02 with the full root-cause report attached. |
| [Feedback Portal idea (OneNote)](https://feedbackportal.microsoft.com/feedback/idea/dd326ebc-9da1-f111-85ce-7c1e529382f4) | Public | Opened 2026-08-27 for the symptom. Root cause added as a comment on 2026-10-02. |
| [Microsoft Q&A thread](https://learn.microsoft.com/en-us/answers/questions/5682454/onenote-sync-says-up-to-date-on-old-pc-but-noteboo) | Public | Answer with the root cause and workaround posted 2026-10-02, for other users with the same symptom. |
| Microsoft Support case | Private | Opened 2026-08-27 and escalated to OneNote engineering. It never found the cause, and Microsoft has not replied since. Full root cause sent 2026-10-02. On 2026-10-07, asked for a case owner and escalation to the Windows Update driver-distribution team and Azure Front Door. Awaiting answer. |

## Samsung

| Channel | Visibility | Status |
|---|---|---|
| [Samsung Community US, Computers board](https://us.community.samsung.com/t5/Computers/Galaxy-Book-ships-Intel-software-that-breaks-IPv6-networking-and/m-p/3679834) | Public | Posted 2026-10-02, hidden by Samsung's spam filter, restored by a moderator. [Update](https://us.community.samsung.com/t5/Computers/Galaxy-Book-ships-Intel-software-that-breaks-IPv6-networking-and/m-p/3683502/highlight/true#M14838) with the October 7 results posted 2026-10-07. |
| Samsung Care live chat (Laptop Department) | Private | Case opened 2026-10-02 and noted for escalation to Galaxy Book software engineering. No follow-up from Samsung. In a second chat on 2026-10-07, Samsung said the first case was "only for documentation" and that it has no software engineering escalation department, and referred the owner to Amazon and a US phone line. A supervisor offered troubleshooting or a repair, then gave the address of Samsung Electronics America's Office of the President. A second case was opened. |
| Samsung Electronics America, Office of the President | Private | Formal complaint and compensation claim sent 2026-10-07, asking Samsung to put the fixed package on Windows Update and to reply in writing within 14 days. Awaiting answer. |

## Not yet received

- Any vendor confirmation that the defect has been reproduced.
- An Intel advisory, or a statement of which ICPS version fixed the flow-label behavior.
- A Samsung, Lenovo, ASUS, HP, VAIO or Dynabook update that replaces the broken builds on Windows Update.
- Any Microsoft statement on Windows Update distribution, WFP flow-label preservation or edge tolerance.
