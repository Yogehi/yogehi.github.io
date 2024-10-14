---
layout: post
title:  "Really Delayed Post Defcon Talk Post"
date:   2024-08-21 9:00:00 -0500
categories: research
tags: random
---

<div align="center">
    <img src="/assets/2024-08-21-really-delayed-post-defcon-talk-post/yaydefconpicyay.jpg">
</div>

Wow we are awesome up on that stage!

In case you didn't know, I did a talk with my friend Ilyes Beghdadi at Defcon 32. It was about our Pwn2Own Toronto 2023 exploit and what Xiaomi did during the event.

<div align="center">
    <img src="/assets/2024-08-21-really-delayed-post-defcon-talk-post/yaydefconscheduleyay.png">
</div>

The people at Zero Day Initiative (they run Pwn2Own) were also excited enough for the talk to actually give us a shoutout during their Pwn2Own Ireland announcement.

<div align="center">
    <img src="/assets/2024-08-21-really-delayed-post-defcon-talk-post/yayzdiyay.png">
</div>

And after the talk, we got some questions that I pretty much half answered. I needed a few days (a lot of a few days) to decompress and see if I could articulate actual good answers for those questions.

So here they are!

### What was the moral of this story? That vendors will try to fuck you over?

lol

The point of Pwn2Own is to show off really cool zero days, while also encouraging vendors to upkeep their security so that they don't get popped at Pwn2Own. As ZDI even said during their Pwn2Own Ireland announcement, they encourage shenanigans. Vendors should be trying to patch as much as possible, and if it happens to fall in line during Pwn2Own, so be it.

But what Xiaomi did was different. They tried to lower their attack surface specifically during Pwn2Own with the intent of removing those limitations after the Pwn2Own event. This means that their interest wasn't securing their devices, but instead just trying to make sure they don't get popped during Pwn2Own.

So the best answer to this question is: **only Xiaomi will try to fuck you over.**

### Pwn2Own seems like a lot of hassle. How do you just keep trying to find these issues while knowing vendors will try to patch them? Doesn't that stress you out?

This ties into one of my favorite stories from working with Ilyes last year on this project.

When we came up with the final PoC for the entry, I literally told Ilyes that he better be ready for the most stressful 2 months of his life.

* Every day you're gonna wake up in fear that something is patched
* Whenever you see an update to the phone, you're gonna panic and hope nothing is patched
* You're gonna want to tell everyone about your exploit, but you gotta keep your mouth shut until after you get your Pwn2Own win
* A week or so before the event, vendors tend to patch shit, so be ready for a week of fucking stress

It's stressful. You lose sleep over potential patches sometimes. You're out hanging out with friends and all you're thinking about is "I wonder if a patch dropped".

Wanna know how I keep going in this environment? **I'm dumb and stubborn**.

There's plenty of reasons to quit doing this shit every year (especially since I'm 1 win 1 loss and 2 "unable to find enough exploits"). But I'm too stubborn to quit.

And that's the thing you'll need if you wanna do Pwn2Own. Being as dumb and stubborn as I am.

### How do I get started in doing Pwn2Own?

Just do it (trademark Nike)

I already answered this in my previous blog post [10 CVEs! My Personal Thoughts On Research And CVEs](https://yogehi.github.io/research/2023/01/04/10-cves-my-personal-thoughts-on-research-and-cves.html):

*Just do it. The only thing stopping most people is their mental block of where they don't know where to start. But the truth is that research has so many places to start, its better to just start in a random place and see where that takes you. And as you get more experienced with research, you'll learn what is considered a "good" starting point, and what is considered a "bad starting point".*

**I think this holds up. I think everyone is too held up on doing "good research" when they should just be worrying about starting research anyway.**

Hell, if you compare my research to some of the shit at Defcon and Blackhat, its shit! There's a shit ton of research out there that makes mine look like shit! People hacked the bootloaders for Samsung phones and found a Windows local privesc that was deemed so good, she got a fucking Pwnie! Good fucking job Chompie!

But the only reason why you're sitting here reading my blog is because I told myself one day "I'm going to buy a Samsung phone and start hacking at it and see what happens".

I guess the only other real answer I can give for this questions is what Ilyes did when he decided he wanted to do Pwn2Own: **research previous vulnerabilities and write ups**. Maybe that will help too!

### Are you still going to do Pwn2Own this year?

I will never publicly disclose my plans until they're forced out of me...such as if you see my name being announced from ZDI.

### Would you hack a Xiaomi device again?

First, I will never responsibly disclose issues to Xiaomi until:

* They stop fucking around at hacking competitions. Pwn2Own isn't the only competition where Xiaomi devices are hacked, and I've heard enough stories about Xiaomi being dicks at these other competitions...
* They increase the payout price for their vulnerabilities
* **They give me my fucking CVE**

Second, because of the above, I will probably not find an incentive to hack a Xiaomi device anytime soon.

**And if I do, I'll need a really really good reason to try to help Xiaomi fix it.**