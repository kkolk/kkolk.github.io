---
layout: post
title:  "Event-stream incident and the need for DevSecOps"
categories: devsecops security
---

Depending on how your build processes work for your in-house applications, if you are using open source frameworks and libraries your dependency tree even for relatively simple applications can depend on a tremendous amount of outside code.  For example even relatively simple Ruby on Rails applications pull in a tremendous amount of code and dependencies.  The Ruby on Rails project itself at the time of this writing has well over 250 outside dependencies.

This complex web of dependencies means that any one piece of software has hundreds of opportunities for a malicious action to attack users of various frameworks and open source projects by embedding malicious code in a single library, then watch it be pulled into the project or projects he's targeting and potentially into the code of a specific company.  The event stream incident just before the holiday season provides a clear example of this type of attack.
<!--more-->

## Rob the bank, from the inside

This particular exploit targeted crytocurrency and specifically a particular wallet application that Bitcoin users use.  This attack began like so many security attacks, with a bit of social engineering.  A developer on Github known as "right9control" volunteered to take over event-stream, the original maintainer who maintains many other projects and wasn't actively working this one handed the project over.  This type of transferring ownership is a common practice in open source, used to help continue projects when the original authors are no longer able or willing to do so.

The project was then updated to include a dependency on another module, flatmap-stream.  This library inclusion wasn't that unusual, these type of additions are common for projects, that's how Ruby on Rails got it's 250 dependencies.  The library even did what it said on the box, it handled a flat map stream. However, it also included an obfuscated snippet that when decrypted reveals Bitcoin-siphoning malware.  The flatmap-stream dependency only existed in event-stream for a short time, then it was again removed.  Event-stream was then released again, this time as a semver-major version bump, which means most applications will not automatically upgrade.  As a result the module most users will pull into their projects will contain the exploit code, but at look at the most current release won't show it.

So how far did the code spread? It turns out event-stream is a very popular package and the version with code was getting roughly 2 million downloads per week.  Before it was discovered it had been downloaded nearly 8 million times.  Luckily this time it was highly targetted towards a particular cryptocurrency wallet product with the aim to steal encryption keys and as a result money from that wallets user. So far the larger community this wasn't as dangerous as i could have been.  Company's didn’t have malicious code running throughout their systems just before the holiday season kicked in full swing.  There is absolutely nothing however that will stop this from occuring again, and there is almost certainly more than one persistant group of attackers trying to do this again.  Potentially with far more damaging results.

