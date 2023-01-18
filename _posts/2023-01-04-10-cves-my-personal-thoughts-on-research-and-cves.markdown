---
layout: post
title:  "10 CVEs! My Personal Thoughts On Research And CVEs"
date:   2023-01-04 10:00:00 -0500
categories: research
tags: random
---

Samsung issued their January 2023 patch, which included 2 more CVEs assigned to me. That makes 10 CVEs so far in my security career.

A coworker told me I should do a talk talking about my CVEs, but I already publish the details of all of my CVEs. I like bragging too much to not publicly disclose *everything*.

So instead, I'll use this platform to say some stuff that I think should be said. I often hear from people trying to get their first CVE, or trying to get into the security field, or just general research.

## How And Why I Research

First, I should mention how I operate when it comes to doing security research:

* I do security research because it is fun and enjoy the chaos that comes with finding shit
* I report my findings because I genuinely get concerned about what I find
* I publicly disclose findings because I like bragging about what I've found

I think the first bullet point is the one thing that needs to be said the loudest:

I DO SECURITY RESEARCH BECAUSE IT IS FUN AND ENJOY THE CHAOS THAT COMES WITH FINDING SHIT

It is this one thing that drives me to get certifications, open up COVID tests, and try again and again at Mobile Pwn2Own (I'm 0/3 attempts now lmao). I genuinely have fun doing what I do. 

And it comes with failures and dead ends. LOTS of dead ends. Last year for Pwn2own 2023, I looked at around 60 Android applications on the Samsung S22, and I only found 2 reportable issues in one single application. This happens. A LOT.

But I look back at the fun I had doing it all, and guess what, I'll be back at it again this year. And that's what I think really needs to be drilled into the people "looking to do research".

*Research has to come from a place of fun and excitement. Otherwise it will be bad and shitty research. Its only when your heart is into the topic that you get good or usable output.*

## My Thoughts On CVEs

They're awesome lmao. Don't let anyone tell you otherwise.

The coworker that asked me to do a talk also said that CVEs are like badges for security researchers.

And they weren't wrong. I'm literally writing a blog post because I got 10 badges to my name. Yeah there's people with a LOT more badges. There's a person at Project Zero who literally did a talk about "Here's all the money Apple owes me because of how many CVEs I have from Apple specifically". If I could find the talk, I would post it here, but I'm writing this in a single morning and I don't feel like going through pages of Google to find the single talk.

But back to my point on this, one thing I think gets lost in the talk about CVEs is that there are people out there who make it their "yearly goal" or something to get at least 1 CVE.

*Literally all of my CVEs were given to me because I was doing what I enjoy. I've never tried to actively look for vulnerabilities that would give me a CVE.*

My first CVE, the Apple DoS bug ([https://yogehi.github.io/cves/cve-2018-4348.html](/cves/cve-2018-4348.html)), I was just randomly looking into macOS mechanics and behaviors with a co-worker. We both had the mindset "Dude macOS is kinda dumb. Let's try to understand why its dumb." We happened to stumble upon the issue during our "lets fuck around with macOS" journey, and we ended up repoting it to Apple because "why not?".

Apple comes back with a "here is a CVE for you" and both of us were excited. Our first CVE! OUR FIRST CVE! YEAH LETS GO OUT AND CELEBRATE! People go through their entire career without getting a CVE but we just got one! Yay!

So what is my personal thought on all of this? What do I hope you, the reader, gets from all of this?

*If you want a CVE, the best thing you can do is just look into stuff you find interesting and fun. Hey look, this goes back directly to the first part of this blog, where you should do research into stuff that you find fun and interesting.*

## Random Questions And Random Answers

I'm going to close this blog out with answering random questions I get about the security field and doing security research.

### How do I get started into security research?

Just do it. The only thing stopping most people is their mental block of where they don't know where to start. But the truth is that research has so many places to start, its better to just start in a random place and see where that takes you. And as you get more experienced with research, you'll learn what is considered a "good" starting point, and what is considered a "bad starting point".

### What kind of software or equipment should I use?

Everyone is different. Google what other people use and experiement with what you're comfortable with.

For me personally, when I do Android research:

* An Android phone (lately its been either the S22 or Pixel 6a)
* Drozer
* Magisk
* Android Studio
* Jadx
* ByteCode Viewer
* Frida / Objection
* XPosed / EdXposed / LSposed
* Windows host OS with Python installed on the host OS
* Ubuntu subsystem on the Windows host
* BurpSuite

### But what if my research doesn't get any good vulnerabilities?

Then you're looking at research from the wrong angle.

Security research is you examining an object built by someone else. You're there to better understand what the object does, how it was coded, and ***IF*** there are any security issues.

Every time I finish a research project, I examine what I did right, what I did wrong, what kind of tooling I could make / have ready for the next project, and what new coding concepts I learned throughout the research. Vulnerabilities just happen to appear once in a while, but only after endless "dead ends" of secure code.

--------------------------------------------------------

I think that's everything I wanted to say in this blog post. And I also suck at ending blog posts.