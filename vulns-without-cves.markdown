---
layout: page
title: Vulns without CVE
permalink: /vulns-without-cves/
---

All of my vulnerabilities that were not assigned a CVE are documented here.

**Wanna learn the skills needed to find these vulns?** I've started making training courses. More information here: [https://yogehi.github.io/training-courses/](https://yogehi.github.io/training-courses/)

--------------------------------------------------------

## Meta Quest System Resource 201.0.0.129 - System-Wide DoS via Unprotected Memory

The Meta Quest "System Resource" diagnostic application (`com.oculus.systemresource`) exposed an unprotected broadcast receiver and service that any local application could abuse to memory-map an 8.6 GB file, exhausting all physical memory and causing a system-wide denial of service.

### References

* [https://yogehi.github.io/vulns-without-cves/2026-meta-quest-system-resource-201.0.0.129-dos/](/vulns-without-cves/2026-meta-quest-system-resource-201.0.0.129-dos/)
* Original post: [https://djini.ai/technical-advisory-meta-quest-system-wide-dos-via-unprotected-memory/](https://djini.ai/technical-advisory-meta-quest-system-wide-dos-via-unprotected-memory/)

--------------------------------------------------------

## Google TV 4.39.2590 - Path Traversal

The Google TV Android application (`com.google.android.videos`) had an exported Content Provider which contains a Path Traversal vulnerability.

### References

* [https://yogehi.github.io/vulns-without-cves/2025-google-tv-4.39.2590-path-traversal/](/vulns-without-cves/2025-google-tv-4.39.2590-path-traversal/)

--------------------------------------------------------
