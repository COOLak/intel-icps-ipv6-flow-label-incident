# Not only Samsung: who ships the affected builds

Research as of 2026-10-07. Every claim below cites its source. "Catalog" means Microsoft's own [Update Catalog](https://www.catalog.update.microsoft.com/Search.aspx?q=pid_icps), which lists the driver packages that partners submit for delivery through Windows Update. The "company" on a Catalog entry is the partner that submitted it.

## The short version

- **Windows Update carries the builds that users have reported as broken, under seven submitters:** Samsung, Lenovo, VAIO, ASUS, HP, Dynabook and the contract manufacturer Huaqin.
- **The Catalog holds 1,086 ICPS entries across 47 versions from 19 companies.** Only **one** entry is the build shown fixed on October 7 (50.26.623.243). It was submitted by Samsung for Windows 11 25H2 and targets device IDs `SID_1001/1002`. The affected Galaxy Book's ICPS device is `SWC\VID_8086&PID_ICPS&SID_0001`, so that entry could not reach it.
- **Every other company's newest Catalog build is older than 50.26.623.243.** Whether those intermediate builds have the bug is unknown: no one has tested them.
- **Independent users report the same IPv6-only resets on HP, ASUS and Lenovo laptops**, and IT admins report that Windows Update puts the software back after they remove it.

## Builds reported broken, and who publishes them on Windows Update

| ICPS build | Status | Evidence | Catalog submitters |
|---|---|---|---|
| 4.1025.207.1 | Reported broken: the Intel service "clean[s] up ipv6 flow table when ssl handshake started"; Azure DevOps resets. Delivered by Windows Update to an HP ZBook Firefly 14 G11. Intel closed the thread twice without a fix. | [Intel Community, Sept 2025](https://community.intel.com/t5/Ethernet-Products/Intel-Connectivity-Network-Service-is-causing-the-ipv6-flowtable/td-p/1716740), [re-post, Nov 2025](https://community.intel.com/t5/Wireless/Intel-Connectivity-Network-Service-is-causing-the-ipv6-flowtable/td-p/1724647) | [HP (6), Dynabook (2)](https://www.catalog.update.microsoft.com/Search.aspx?q=4.1025.207.1) |
| 40.25.725.165 | Reported broken: 40–60% of IPv6 connections to AWS CloudFront reset on an ASUS Vivobook S 14 S5406SA; none after removing ICPS. Intel: "we do not have a confirmed public advisory for this specific behavior". | [Intel Community, July 2026](https://community.intel.com/t5/Wireless/Intel-Connectivity-Performance-Suite-causes-intermittent-IPv6/td-p/1754394), [Intel's reply](https://community.intel.com/t5/Wireless/Intel-Connectivity-Performance-Suite-causes-intermittent-IPv6/m-p/1754422) | [ASUS (30), HP (3), Lenovo (3), VAIO (3)](https://www.catalog.update.microsoft.com/Search.aspx?q=40.25.725.165) |
| 40.25.926.173 (ICPS 40.25.926.0, driver 11.5.11.19) | Broken: packet captures and controlled tests on this tracker's Galaxy Book | [Technical analysis](technical-analysis.md) | [Lenovo (12), VAIO (4), Huaqin (4), Samsung (3)](https://www.catalog.update.microsoft.com/Search.aspx?q=40.25.926.173) |
| 50.26.623.243 (driver 12.10.14.33) | **Fixed**, in this tracker's October 7 test. Intel's release notes say only "minor fixes that improve performance, stability, and vendor-specific features". | [October 7 test](technical-analysis.md#october-7-reproduced-again-and-intels-generic-build-does-not-have-the-bug), [release notes](https://downloadmirror.intel.com/927067/ReleaseNotes_IntelConnectivityPerformanceSuite_50.26.623.243.pdf) | [Samsung (1), 25H2, SID_1001/1002 only](https://www.catalog.update.microsoft.com/ScopedViewInline.aspx?updateid=e79f0755-ec7f-4cb6-b6e4-ed8fad7096a5) |

## By laptop maker

| Maker | Ships ICPS | Newest build in the Catalog / on the maker's site | Independent reports |
|---|---|---|---|
| ASUS | Yes, [529 Catalog entries](https://www.catalog.update.microsoft.com/Search.aspx?q=Connectivity%20Performance%20Suite%20ASUSTeK), the most of any company | 50.26.220.212. ASUS's own driver pages still list 40.25.725.165 as newest for the [Zenbook 14 UX3405MA](https://www.asus.com/support/api/product.asmx/GetPDDrivers?cpu=&osid=52&website=global&pdhashedid=&model=UX3405MA) and [Vivobook S 14 S5406SA](https://www.asus.com/support/api/product.asmx/GetPDDrivers?cpu=&osid=52&website=global&pdhashedid=&model=S5406SA) | Vivobook S 14 (above). Windows Update [reinstalled 40.25.725.165 within about two days](https://github.com/safing/portmaster/issues/2120) on a Zenbook 14 after removal |
| HP | Yes, 130 Catalog entries | [50.26.401.224](https://www.catalog.update.microsoft.com/ScopedViewInline.aspx?updateid=4e072327-653d-4c9f-b390-00d6d9468bf6). HP's [SoftPaq sp157565](https://ftp.hp.com/pub/softpaq/sp157501-158000/sp157565.html) ships 4.1025.207.1 for 32 models | ZBook Firefly 14 G11 (above). An organization with [EliteBook X Flip G1i and EliteBook 8 G1i](https://community.intel.com/t5/Wireless/Intel-Connectivity-Network-Service-is-causing-the-ipv6-flowtable/m-p/1750798): Microsoft 365 Search and SharePoint fail over IPv6. Another EliteBook 8 G1i owner: "[it took just one Windows Update and the problem was back](https://community.intel.com/t5/Wireless/Intel-Connectivity-Network-Service-is-causing-the-ipv6-flowtable/m-p/1750920)" |
| Lenovo | Yes, 61 Catalog entries, including 12 of the broken 40.25.926.173 | [50.25.1121.193](https://www.catalog.update.microsoft.com/ScopedViewInline.aspx?updateid=00bb4184-a33a-44da-b696-f741f187f086) for Windows 11; 40.25.926.173 is still newest for Windows 10. Lenovo's site offers [40.25.829.165](https://pcsupport.lenovo.com/us/en/downloads/ds570299) for Yoga 7 2-in-1 and Yoga Pro 7 | [Several new Lenovo laptops, out of the box](https://community.brave.app/t/err-connection-reset-errors-on-new-win-11-laptops/637663): resets fixed by disabling IPv6 or the Intel service. Lenovo separately documents an [ICPS defect that breaks Outlook, Teams and OneDrive with VPNs](https://pcsupport.lenovo.com/us/en/solutions/ht515468) |
| VAIO | Yes, 33 Catalog entries | [50.25.1121.193](https://www.catalog.update.microsoft.com/ScopedViewInline.aspx?updateid=11b4b528-c9e5-42b1-9010-8af0b5a20612); VAIO's site publishes a [40.25.725.165 update program](https://support.us.vaio.com/knowledge-base/intel-connectivity-performance-suite-ver-40-25-725-165-update-program/) | None found |
| Dynabook | Yes, 64 Catalog entries | [50.26.522.234](https://www.catalog.update.microsoft.com/ScopedViewInline.aspx?updateid=03ec1b1c-377c-4cd4-ba85-b1ff0c58dce5) for 25H2; [40.25.909.171](https://www.catalog.update.microsoft.com/Search.aspx?q=40.25.909.171) for 24H2 and Windows 10 | None found |
| Samsung | Yes, 80 Catalog entries | The one fixed entry (SID_1001/1002, 25H2); [50.26.107.203](https://www.catalog.update.microsoft.com/ScopedViewInline.aspx?updateid=369bcb5f-50b7-4901-bd34-8488d9cb459c) for SID_0001/0002 | This tracker |
| Dell | Yes: "[ICPS is intended for all Dell laptops that have an Intel CPU](https://www.dell.com/support/kbdoc/en-us/000275594/how-to-get-support-for-icps-and-icm-on-a-dell-computer)" | No Dell-submitted Catalog entries; version unknown | None in the ICPS era. The predecessor Killer software caused the same IPv6-only resets on [Alienware and XPS machines in 2021–22](https://community.intel.com/t5/Wireless/Killer-Network-Service-preventing-connection-to-some-websites/td-p/1366144) |
| Fujitsu | Yes, on vPro models; Windows Update reinstalls it after removal (Fujitsu's own support FAQ) | Version unknown | None found |
| Others | Getac, Panasonic, Microsoft Surface (2.x builds), Huawei, LG, and the contract manufacturers Quanta, Compal, Emdoor, PEGATRON, Foxconn | All older than 50.26.623.243 ([Catalog](https://www.catalog.update.microsoft.com/Search.aspx?q=pid_icps)) | None found |

## IT admins

- A public Intune remediation script [removes ICPS because of IPv6 `ERR_CONNECTION_RESET`](https://github.com/realmjoin/realmjoin-remediation/tree/main/intel/remove-intel-connectivity-performance-suite). It leaves the drivers alone, because "Windows Update installs them again". It probably shares an author with the HP organization report above.
- In the Killer era, organizations running Dell fleets reported the same Microsoft-only resets on [Microsoft Q&A](https://learn.microsoft.com/en-us/answers/questions/4340112/random-connection-reset-error).

## Caveats

- This tracker's captures are the only public packet-level proof of the flow label being zeroed. The HP report describes the same mechanism in words; the ASUS report measured the resets but not the label.
- Only three builds are reported broken. More than ten other builds have never been tested either way, so "older than the fix" does not mean "affected".
- A Catalog entry does not show which laptop models Windows Update offers it to.
- No device counts exist anywhere.
