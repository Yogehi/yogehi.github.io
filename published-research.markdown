---
layout: page
title: Published Research
permalink: /published-research/
---

All of my research which were published on other sites are documented here.

## Faking A Positive COVID Test

I conducted research into the Ellume COVID-19 Home Test with the intention of finding methods to fake a COVID test result. This device was chosen specifically because of the Bluetooth device that is used as the analyzer for testing a nasal sample. As for the outcome of this research, F-Secure was successful in falsifying a COVID test result, and obtained a certificate verifying the COVID test result. This article will go over the technical details of how this research was conducted.

Full write up: [https://labs.f-secure.com/blog/faking-a-positive-covid-test](https://labs.f-secure.com/blog/faking-a-positive-covid-test)

News articles:

* [https://www.engadget.com/ellume-covid-19-test-bluetooth-hack-153933652.html](https://www.engadget.com/ellume-covid-19-test-bluetooth-hack-153933652.html)
* [https://www.theverge.com/2021/12/21/22847222/ellume-at-home-covid-test-bluetooth-android-certification](https://www.theverge.com/2021/12/21/22847222/ellume-at-home-covid-test-bluetooth-android-certification)
* [https://techcrunch.com/2021/12/21/ellume-bug-covid-results](https://techcrunch.com/2021/12/21/ellume-bug-covid-results)

--------------------------------------------------------

## Samsung S20 - RCE via Samsung Galaxy Store App

I looked into exploiting the Samsung S20 device for Tokyo Pwn2Own 2020. An exploit chain was found for version 4.5.19.13 of the Galaxy Store application that could have allowed an attacker to install any application on the Galaxy Store without user consent. Samsung patched this vulnerability at the end of September 2020, no longer making it a viable entry for Pwn2Own. This blog post went over the technical details of this vulnerability and how I intended on exploiting this issue for Pwn2Own before it was patched. 

Full write up: [https://labs.f-secure.com/blog/samsung-s20-rce-via-samsung-galaxy-store-app/](https://labs.f-secure.com/blog/samsung-s20-rce-via-samsung-galaxy-store-app/)

I also created a Docker image to more easily exploit this issue: [https://github.com/Yogehi/pwn2own2020-mitmInstallApp-docker](https://github.com/Yogehi/pwn2own2020-mitmInstallApp-docker)

--------------------------------------------------------

## Uncommon SQL Database Alert - Informix SQL Injection

A client was looking to upgrade their Cisco UCM software and wanted assurance that their implementation was configured securely. During the assessment, we had discovered an authenticated SQL Injection issue within the Cisco UCM administrator portal. This research goes over the process of discovering how Informix SQL works and developing a custom script to exploit this issue.

Full write up: [https://labs.f-secure.com/blog/uncommon-sql-database-alert-informix-sql-injection/](https://labs.f-secure.com/blog/uncommon-sql-database-alert-informix-sql-injection/)