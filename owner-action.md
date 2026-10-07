# What each vendor needs to do

This is a completed root-cause analysis with controlled A/B proof, not a request for basic troubleshooting. Sign-out, cache resets, Office repair, reinstalling and "use the web version" are all irrelevant to this defect. Please route it straight to networking or platform engineering.

> **Update, October 7, 2026:** Intel's current generic ICPS 50.26.623.243 (driver `IntcCo11X64.sys` 12.10.14.33) does not have the bug. The OEM build 40.25.926.173 does, and it reached the affected laptop through Windows Update. So the fix exists. What's missing is getting it to the people who have the old build. The asks below are updated accordingly.

## Intel (Connectivity Performance Suite / Rivet traffic-control engine)

0. **Say which version fixed it**, and get that build to every OEM that ships ICPS and onto Windows Update. Publish an advisory so that users with IPv6 connection resets can find the cause, instead of being told to uninstall ICPS.
1. **Fix the callout** (done in the generic build, per the October 7 test). When the ICPS traffic-control callout absorbs and re-injects (or clones) an IPv6 packet, it must keep the original Flow Label. Copy it from the original `NET_BUFFER_LIST` or the flow's endpoint, and never emit `0` partway through a flow. The defect is in `IpV6 Egress Transport Callout` (`FWPM_LAYER_OUTBOUND_TRANSPORT_V6`, `FWP_CALLOUT_FLAG_CONDITIONAL_ON_FLOW`) and/or `IpV6 Ingress Network Outbound Callout` (`FWPM_LAYER_OUTBOUND_IPPACKET_V6`) in `IntcCo11X64.sys` 11.5.11.19.
2. **Ship it** through Windows Update and OEM channels, and **publish an advisory**. Affected users are having IPv6 connections silently reset with no way to know why.
3. **Until it is fixed, disable the IPv6 traffic-control path** in OEM images and updates.
4. **Stop doing this invisibly.** An always-on kernel-level packet rewriter that the owner never asked for, can't see and can't control should not be in the network path of every connection.

## Samsung (Galaxy Book factory image and Samsung Update)

1. **Replace the 40.25.926.173 ICPS packages** that Samsung publishes through Windows Update and Samsung Update with a fixed build (50.26.623.243 or later), or pull them. Intel's fixed build exists; Galaxy Book owners are still being sent the broken one.
2. **Push a fix or a disabled configuration to existing owners.** Nobody will diagnose this on their own. Samsung's own tools (Device care, Galaxy Book Experience, Samsung Update) neither detect nor mention it.
3. **Review what gets preinstalled.** Kernel-level network "optimizers" belong on a laptop only if they are visible, optional and tested against mainstream services.

## Microsoft

**Windows Update / Hardware Dev Center**

- Windows Update delivered the broken ICPS 40.25.926.173 extension and component to the affected laptop on 2026-05-09, 2026-05-14 and again on 2026-06-13. Stop offering those submissions, or require them to be superseded by a fixed build. They break Microsoft's own services over IPv6 on the machines that receive them.

**Windows networking / Windows Filtering Platform**

- Confirm whether the WFP transport-layer injection path (`FwpsInjectTransportSendAsync` and the clone/re-inject pattern third-party callouts use) preserves the IPv6 Flow Label. If it doesn't, the stack should preserve or re-derive the endpoint's label on injection. Otherwise every third-party callout that re-injects will break flow-label-hashing load balancers, Microsoft's own included.

**Azure Front Door and the OneDrive/OneNote service owners**

- The edge's IPv6 load balancing turns one non-conformant, but Intel-signed and OEM-preinstalled, client driver into a hard outage for OneNote, OneDrive and Microsoft 365. It should tolerate a flow label that changes within a connection, as Google, Cloudflare and Akamai demonstrably do. Options include 5-tuple-consistent hashing, or SYN-cookie/state sharing between nodes.

**OneNote**

- A connection reset is reported as "We can't get to your notebooks right now". Worse, a notebook that never downloaded shows a false "Up to date".
- There is no IPv4 fallback after an IPv6 reset, and no diagnosable error is surfaced to the user.
- Customers lose weeks of sync and get told to sign out and reinstall, which risks data loss. An escalated Microsoft support case opened in August 2026 never found the cause.
