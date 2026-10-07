# Am I affected? Check and fix

This applies to Windows PCs with **Intel Connectivity Performance Suite** installed. It usually ships preinstalled on Intel laptops, and it is most visible on **native IPv6** connections.

## Typical symptoms

- OneNote desktop: "We can't get to your notebooks right now", or notebooks open empty while showing "Up to date".
- OneDrive or Microsoft 365 sign-in failing intermittently. `ERR_CONNECTION_RESET` in the browser.
- "Connection reset" on assorted other websites.
- Everything works on the phone or in OneNote on the web.

## 1. Check whether the Intel driver is present

Open **PowerShell** and run:

```powershell
Get-CimInstance Win32_SystemDriver -Filter "Name='INTCCoSvc'" | Select-Object Name, StartMode, State
Get-Service IDBWM, 'Intel Connectivity Network Service', IntelConnectService -ErrorAction SilentlyContinue
```

If `INTCCoSvc` shows `State: Running`, the traffic-control driver is active.

## 2. Test IPv6 connections to Microsoft's edge

```powershell
1..10 | ForEach-Object { curl.exe -6 -sS -o NUL -w "%{http_code}`n" https://api.onedrive.com/ }
```

A healthy machine prints ten HTTP status codes, such as `302`. An affected machine prints errors like `Recv failure: Connection was reset` for some or all of the attempts. If `curl -6` can't connect at all, you have no IPv6, and this particular bug can't hit you.

## 3. Apply the workaround

Open **PowerShell as administrator** and run:

```powershell
sc.exe config INTCCoSvc start= disabled
sc.exe stop INTCCoSvc
foreach ($s in 'IDBWM', 'Intel Connectivity Network Service', 'IntelConnectService') {
  Set-Service -Name $s -StartupType Disabled -ErrorAction SilentlyContinue
  Stop-Service -Name $s -Force -ErrorAction SilentlyContinue
}
```

Restart the PC, then run step 2 again. All ten attempts should succeed. OneNote should sync within a minute. If a notebook still shows an error, close it and reopen it from OneDrive.

**To undo it**, restore the start types they had on the affected machine, then restart:

```powershell
sc.exe config INTCCoSvc start= demand
Set-Service IDBWM -StartupType Manual
Set-Service 'Intel Connectivity Network Service' -StartupType Automatic
Set-Service IntelConnectService -StartupType Manual
```

Intel Connectivity Performance Suite only provides Intel's traffic prioritization features. Wi-Fi and Bluetooth keep working without it.

## 4. Or replace the OEM build with Intel's current generic build

Tested on 2026-10-07 on the affected laptop: Intel's generic **ICPS 50.26.623.243** (traffic-control driver `IntcCo11X64.sys` 12.10.14.33) does **not** zero the flow label. With it installed and running, 169 of 170 IPv6 connections succeeded and every packet kept its label. See [the test](technical-analysis.md#october-7-reproduced-again-and-intels-generic-build-does-not-have-the-bug).

1. Check which version you have: `pnputil /enum-drivers | Select-String icps -Context 1,3`. Versions 40.x and 4.1025.x are the OEM builds that showed the bug.
2. Remove the OEM build the way Intel's support article [000093451](https://www.intel.com/content/www/us/en/support/articles/000093451/wireless/wireless-software.html) describes: in an administrator PowerShell, run `pnputil /delete-driver oemNN.inf /uninstall` for each `icpscomponent.inf` and then the `icpsextension.inf` that step 1 listed. Then remove the "Intel Connectivity Performance Suite" app (Settings > Apps) and restart. Your Wi-Fi drops for a moment while the extension is removed. Don't remove any other driver package.
3. Download **IntelConnectivityPerformanceSuite-50.26.623.243.exe** from [Intel's download page](https://www.intel.com/content/www/us/en/download/738623/intel-connectivity-performance-suite-icps-for-intel-wireless-products-and-intel-connectivity-manager-icm.html), check its SHA-256 against the value Intel lists, and install it.
4. Run the test in step 2 above again.

If Intel's installer reports a failed uninstallation of an old version, Intel's download page points to its ICPS Clean Uninstall Utility.

## Things that do not help

Signing out of Office, clearing the OneNote cache, repairing or reinstalling Office, resetting the network stack and switching DNS do nothing for this. Turning IPv6 off hides the symptom, but it also disables IPv6 for everything else.

## If you are affected

Please add a 👍 or a short comment to the [tracking issue](https://github.com/COOLak/intel-icps-ipv6-flow-label-incident/issues/1) with your laptop model, ICPS version and Windows build. Independent reports help the vendors reproduce it. Don't post IP addresses, email addresses or ticket numbers.
