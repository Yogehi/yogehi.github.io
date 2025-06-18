---
layout: page
title: Published Research
permalink: /published-research/
---

All of my research which were published on other sites are documented here.

**Wanna learn the skills used in some of the research below?** I've started making training coureses. More information here: [https://yogehi.github.io/training-courses/](https://yogehi.github.io/training-courses/)

--------------------------------------------------------

## OffensiveCon25 - Chainspotting 2: The Unofficial Sequel to the 2018 Talk “Chainspotting”

At Pwn2Own Ireland 2024 (sometimes referred to as Mobile Pwn2Own 2024), there were 61 entries targeting...IoT devices and printers. No wonder "mobile" is not in the event's title anymore. Thankfully, there was still 1 entry that targeted, and successfully pwned, the Samsung Galaxy S24. And now that the issues are patched, it is time to disclose those technical details!

The full exploit chain consisted of five different issues across several different applications, resulting in the ability to install arbitrary APKs. This talk will discuss the bugs that were discovered, how they were chained together, and the issues encountered while developing the Pwn2Own entry.

There are no stories about vendors being lame this year. Just pure technical details about the bugs, and how a ""Path Traversal"" issue ended up being the most interesting bug in the entire exploit chain.

OffensiveCon25 Assets:

* Slide Deck: [NCC Group Blog](https://www.nccgroup.com/media/smbhcb24/offensivecon-2024-talk-chainspotting-2-censored-but-for-real.pdf)
    * Backup Slide Deck: [https://yogehi.github.io/assets/offensivecon25-talk-stuff/OffensiveCon 2024 Talk - Chainspotting 2.pdf](/assets/offensivecon25-talk-stuff/OffensiveCon%202024%20Talk%20-%20Chainspotting%202.pdf)
* YouTube Video: [https://www.youtube.com/watch?v=LAIr2laU-So](https://www.youtube.com/watch?v=LAIr2laU-So)

Links:

* White Paper: [https://www.nccgroup.com/us/research-blog/samsung-galaxy-s24-pwn2own-ireland-2024/](https://www.nccgroup.com/us/research-blog/samsung-galaxy-s24-pwn2own-ireland-2024/)
    * Backup White Paper - [https://yogehi.github.io/published-research/pwn2own-ireland-2024-samsung-s24-attack-chain](/published-research/pwn2own-ireland-2024-samsung-s24-attack-chain)
* Pwn2Own Success Tweet - [https://x.com/thezdi/status/1849022145512230983](https://x.com/thezdi/status/1849022145512230983)
* YouTube Short - [https://www.youtube.com/shorts/eM9dOhHH2AA](https://www.youtube.com/shorts/eM9dOhHH2AA)

--------------------------------------------------------

## DEF CON 32 Talk - Xiaomi The Money - Our Toronto Pwn2Own Exploit and Behind The Scenes Story

At Pwn2Own Toronto 2023, NCC Group was one of the two teams that compromised the Xiaomi 13 Pro. The exploit chain involved using a malicious HTML hyperlink and uploading a potentially malicious application to the Xiaomi app store.

However, this talk is not just about the technical details of the exploit. While researching the final exploit, NCC Group discovered how an exploit could work in one region of the world, but not in other regions, and how the researchers had to travel to Canada for a day just to test if the exploit would work in Canada. This talk also discusses just how far Xiaomi is willing to go to make sure their device isn't hacked at Pwn2Own, and why only two teams were able to successfully compromise the device during the competition.

DEF CON Assets:

* Slide Deck: [DEFCON 32 Media Server / DEF CON 32 / DEF CON 32 presentations / Presentation.pdf](https://media.defcon.org/DEF%20CON%2032/DEF%20CON%2032%20presentations/DEF%20CON%2032%20-%20Ken%20Gannon%20Ilyes%20Beghdadi%20-%20Xiaomi%20The%20Money%20Our%20Toronto%20Pwn2Own%20Exploit%20and%20Behind%20The%20Scenes%20Story.pdf)
    * Backup Slide Deck: [https://yogehi.github.io/assets/defcon-32-talk-stuff/DEF CON 32 - Ken Gannon Ilyes Beghdadi - Xiaomi the money Our Toronto Pwn2own Exploit and Behind The Scenes Story.pdf](/assets/defcon-32-talk-stuff/DEF%20CON%2032%20-%20Ken%20Gannon%20Ilyes%20Beghdadi%20-%20Xiaomi%20The%20Money%20Our%20Toronto%20Pwn2Own%20Exploit%20and%20Behind%20The%20Scenes%20Story.pdf)
* Exploit Video: [DEFCON 32 Media Server / DEF CON 32 / DEF CON 32 presentations / Exploit.mp4](https://media.defcon.org/DEF%20CON%2032/DEF%20CON%2032%20presentations/DEF%20CON%2032%20-%20Ken%20Gannon%20Ilyes%20Beghdadi%20-%20Xiaomi%20The%20Money%20Our%20Toronto%20Pwn2Own%20Exploit%20and%20Behind%20The%20Scenes%20Story-exploit.mp4)
    * Backup Exploit Video: [httpS;//yogehi.github.io/assets/defcon-32-talk-stuff/DEf cON 32 - ken Gannon Ilyes Beghdadi - Xiaomi The Money Our Toronto Pwn2Own Exploit and Behind The Scenes Story-exploit.mp4](/assets/defcon-32-talk-stuff/DEF%20CON%2032%20-%20Ken%20Gannon%20Ilyes%20Beghdadi%20-%20Xiaomi%20The%20Money%20Our%20Toronto%20Pwn2Own%20Exploit%20and%20Behind%20The%20Scenes%20Story-exploit.mp4)
* Talk MP4 Video: [DEFCON 32 Media Server / DEF CON 32 / DEF CON 32 video and slides / Talk.mp4](https://media.defcon.org/DEF%20CON%2032/DEF%20CON%2032%20video%20and%20slides/DEF%20CON%2032%20-%20Xiaomi%20The%20Money%20-%20Our%20Toronto%20Pwn2Own%20Exploit%20and%20BTS%20Story%20-%20Ken%20Gannon%2C%20Ilyes%20Beghdadi.mp4)
    * Backup Talk Video: [https://yogehi.github.io/assets/defcon-32-talk-stuff/DEF CON 32 - Xiaomi The Money - Our Toronto Pwn2Own Exploit and BTS Story - Ken Gannon, Ilyes Beghdadi.mp4](/assets/defcon-32-talk-stuff/DEF%20CON%2032%20-%20Xiaomi%20The%20Money%20-%20Our%20Toronto%20Pwn2Own%20Exploit%20and%20BTS%20Story%20-%20Ken%20Gannon,%20Ilyes%20Beghdadi.mp4)
* Talk YouTube Video: [https://www.youtube.com/watch?v=B0A8F_Izmj0](https://www.youtube.com/watch?v=B0A8F_Izmj0)

Links: 

* Advisory: [https://www.nccgroup.com/us/research-blog/technical-advisory-xiaomi-13-pro-code-execution-via-getapps-dom-cross-site-scripting-xss/](https://www.nccgroup.com/us/research-blog/technical-advisory-xiaomi-13-pro-code-execution-via-getapps-dom-cross-site-scripting-xss/)
    * Backup advisory - [https://yogehi.github.io/cves/cve-2024-4406.html](/cves/cve-2024-4406.html)
* Pwn2Own Success Tweet - [https://x.com/thezdi/status/1716936539345936682](https://x.com/thezdi/status/1716936539345936682)
* YouTube Short - [https://www.youtube.com/shorts/WD1OgZI8Kh4](https://www.youtube.com/shorts/WD1OgZI8Kh4)

--------------------------------------------------------

## Faking Another Positive COVID Test

I conducted research into the Cue COVID-19 Home Test with the intention of finding methods to fake a COVID test result. This device was chosen specifically because of the Bluetooth device that is used as the analyzer for testing a nasal sample. As for the outcome of this research, WithSecure was successful in falsifying a COVID test result, and obtained a certificate verifying the COVID test result. This article will go over the technical details of how this research was conducted.

Links:

* Full write up: [https://labs.f-secure.com/blog/faking-another-positive-covid-test/](https://labs.f-secure.com/blog/faking-another-positive-covid-test/)
    * Backup write up: [https://yogehi.github.io/published-research/faking-another-positive-covid-test.html](/published-research/faking-another-positive-covid-test.html)

Press release:

* [https://www.withsecure.com/en/whats-new/pressroom/withsecure-and-cue-health-collaborate-to-strengthen-the-integrity-of-covid-19-test-results](https://www.withsecure.com/en/whats-new/pressroom/withsecure-and-cue-health-collaborate-to-strengthen-the-integrity-of-covid-19-test-results)

News articles:

* [https://www.pcmag.com/news/flaw-in-covid-19-testing-gadget-couldve-been-exploited-to-change-results](https://www.pcmag.com/news/flaw-in-covid-19-testing-gadget-couldve-been-exploited-to-change-results)
* [https://techcrunch.com/2022/04/21/cue-health-covid-security-false-results/](https://techcrunch.com/2022/04/21/cue-health-covid-security-false-results/)

--------------------------------------------------------

## Faking A Positive COVID Test

I conducted research into the Ellume COVID-19 Home Test with the intention of finding methods to fake a COVID test result. This device was chosen specifically because of the Bluetooth device that is used as the analyzer for testing a nasal sample. As for the outcome of this research, F-Secure was successful in falsifying a COVID test result, and obtained a certificate verifying the COVID test result. This article will go over the technical details of how this research was conducted.

Links:

* Full write up: [https://labs.f-secure.com/blog/faking-a-positive-covid-test](https://labs.f-secure.com/blog/faking-a-positive-covid-test)
    * Backup write up: [https://yogehi.github.io/published-research/faking-a-positive-covid-test.html](/published-research/faking-a-positive-covid-test.html)

Press release:

* [https://www.f-secure.com/en/press/p/f-secure-researcher-helps-improve-integrity-of-ellume-covid-19-h](https://www.f-secure.com/en/press/p/f-secure-researcher-helps-improve-integrity-of-ellume-covid-19-h)

News articles:

* [https://www.engadget.com/ellume-covid-19-test-bluetooth-hack-153933652.html](https://www.engadget.com/ellume-covid-19-test-bluetooth-hack-153933652.html)
* [https://www.theverge.com/2021/12/21/22847222/ellume-at-home-covid-test-bluetooth-android-certification](https://www.theverge.com/2021/12/21/22847222/ellume-at-home-covid-test-bluetooth-android-certification)
* [https://techcrunch.com/2021/12/21/ellume-bug-covid-results](https://techcrunch.com/2021/12/21/ellume-bug-covid-results)

--------------------------------------------------------

## Samsung S20 - RCE via Samsung Galaxy Store App

I looked into exploiting the Samsung S20 device for Tokyo Pwn2Own 2020. An exploit chain was found for version 4.5.19.13 of the Galaxy Store application that could have allowed an attacker to install any application on the Galaxy Store without user consent. Samsung patched this vulnerability at the end of September 2020, no longer making it a viable entry for Pwn2Own. This blog post went over the technical details of this vulnerability and how I intended on exploiting this issue for Pwn2Own before it was patched. 

Links:

* Full write up: [https://labs.f-secure.com/blog/samsung-s20-rce-via-samsung-galaxy-store-app/](https://labs.f-secure.com/blog/samsung-s20-rce-via-samsung-galaxy-store-app/)
    * Backup write up: [https://yogehi.github.io/published-research/samsung-s20-rce-galaxy-store.html](/published-research/samsung-s20-rce-galaxy-store.html)

I also created a Docker image to more easily exploit this issue: [https://github.com/Yogehi/pwn2own2020-mitmInstallApp-docker](https://github.com/Yogehi/pwn2own2020-mitmInstallApp-docker)

--------------------------------------------------------

## Uncommon SQL Database Alert - Informix SQL Injection

A client was looking to upgrade their Cisco UCM software and wanted assurance that their implementation was configured securely. During the assessment, we had discovered an authenticated SQL Injection issue within the Cisco UCM administrator portal. This research goes over the process of discovering how Informix SQL works and developing a custom script to exploit this issue.

Links: 

* Full write up: [https://labs.f-secure.com/blog/uncommon-sql-database-alert-informix-sql-injection/](https://labs.f-secure.com/blog/uncommon-sql-database-alert-informix-sql-injection/)
    * Backup write up: [https://yogehi.github.io/published-research/informix-sql-injection.html](/published-research/informix-sql-injection.html)