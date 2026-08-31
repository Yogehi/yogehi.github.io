---
layout: page
title: Vulns without CVE
permalink: /vulns-without-cves/
---

All of my vulnerabilities that were not assigned a CVE are documented here.

**Wanna learn the skills needed to find these vulns?** I've started making training courses. More information here: [https://yogehi.github.io/training-courses/](https://yogehi.github.io/training-courses/)

--------------------------------------------------------

## Meta Horizon Shell 201.0.0.649.547 - Automatic Dangerous Permission Grant via Virtual Input Injection

`GauntletTestPanelActivity` in Meta Horizon Shell (`com.oculus.vrshell`) registered two dynamic broadcast receivers with no sender permission check, letting any installed application inject arbitrary virtual key/mouse events via `VirtualInputDeviceManager`. This let a malicious app automatically navigate and accept all dangerous permission dialogs, silently granting itself camera, microphone, and storage access with zero user interaction.

### References

* [https://djini.ai/technical-advisory-meta-horizon-shell-automatic-dangerous-permission-grant-via-virtual-input-injection/](https://djini.ai/technical-advisory-meta-horizon-shell-automatic-dangerous-permission-grant-via-virtual-input-injection/)
    * Backup Post: [https://yogehi.github.io/vulns-without-cves/2026-meta-horizon-shell-201.0.0.649.547-permission-grant/](/vulns-without-cves/2026-meta-horizon-shell-201.0.0.649.547-permission-grant/)

--------------------------------------------------------

## Samsung Dressroom (Wallpaper & Style) 2.9.50.89 - Arbitrary File Write at System UID

The Samsung Wallpaper & Style / Dressroom app (`com.samsung.android.app.dressroom`) extracted a nested, attacker-supplied ZIP archive during Smart Switch backup restore without any path containment, and neutralized the platform's `ZipPathValidator` defense. This allowed a Zip-Slip arbitrary file write as UID 1000, which could be escalated to bypass `WRITE_SECURE_SETTINGS` entirely by injecting a malicious accessibility service into `settings_secure.xml`.

### References

* [https://djini.ai/technical-advisory-samsung-dressroom-wallpaper-style-arbitrary-file-write-at-system-uid/](https://djini.ai/technical-advisory-samsung-dressroom-wallpaper-style-arbitrary-file-write-at-system-uid/)
    * Backup Post: [https://yogehi.github.io/vulns-without-cves/2026-samsung-dressroom-2.9.50.89-arbitrary-file-write/](/vulns-without-cves/2026-samsung-dressroom-2.9.50.89-arbitrary-file-write/)

--------------------------------------------------------

## Meta Quest System Resource 201.0.0.129 - System-Wide DoS via Unprotected Memory

The Meta Quest "System Resource" diagnostic application (`com.oculus.systemresource`) exposed an unprotected broadcast receiver and service that any local application could abuse to memory-map an 8.6 GB file, exhausting all physical memory and causing a system-wide denial of service.

### References

* [https://djini.ai/technical-advisory-meta-quest-system-wide-dos-via-unprotected-memory/](https://djini.ai/technical-advisory-meta-quest-system-wide-dos-via-unprotected-memory/)
    * Backup Post: [https://yogehi.github.io/vulns-without-cves/2026-meta-quest-system-resource-201.0.0.129-dos/](/vulns-without-cves/2026-meta-quest-system-resource-201.0.0.129-dos/)

--------------------------------------------------------

## Google TV 4.39.2590 - Path Traversal

The Google TV Android application (`com.google.android.videos`) had an exported Content Provider which contains a Path Traversal vulnerability.

### References

* [https://yogehi.github.io/vulns-without-cves/2025-google-tv-4.39.2590-path-traversal/](/vulns-without-cves/2025-google-tv-4.39.2590-path-traversal/)

--------------------------------------------------------
