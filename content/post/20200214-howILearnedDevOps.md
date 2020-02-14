---
title: "How I learned DevOps"
date: 2020-02-14
tags: [devops]
draft: false
---

This post is basically how I learned to DevOps. It's not "How to Become a DevOps Engineer". There is no one answer. I had a lot of luck and a lot of passion. Instead, this is about my career path, and how I ended up the Lead SRE of a multi-national company.

First of all, I love CLI tools. My everyday toolkit is `zsh`, `vim`, `grep`, `ssh`, and `git`. I learned all these at my first job working on custom hardware. At the time, I was a new CS grad with a go-get-em attitude towards proper software design and building apps. The iPhone had just come out, and I was sure I'd work on apps for the rest of my career. Side note: I have never published a single app.

The job wasn't like that. Instead, I wrote kernel modules and learned all about `/var/logs`. Grep became indispensable. I wrote GTK+ widgets in object oriented C (not Objective-C), and that is the worst thing I've done in my career. On the plus side, I learned how to read some Chinese due to all the helpful Google results on GTK+ being Chinese forums.

Linux became second nature and its low level machinery was no longer a mystery. My very own patch was accepted into the kernel to fix a `ioctl` call that was causing our systems to crash. Even more insane, I wrote a complex build/deployment system in bash that required translating a home brew `cmake` like build system from Perl to Python.

One day, Google contacted me for a job interview. I passed the initial foot-in-the-door part of the process, and the recruiter asked me where I'd like to apply to within the company. I had no clue. I thought maybe Android? She asked me to describe my abilities and current job. Without hesitation, she suggested the Site Reliability Engineering group, which I had no idea what that was.

Sadly (or gladly), I didn't get the job at Google. Instead, I took a year off to teach myself about modern application design. I built microservices, messed around with these things called Docker, started learning the then brand new React framework. After a bunch of interviews, I ended up a mid-level QA Automation Engineer for a local mid-stage startup.

There was good year of work there in the automation role before things started to sour. I gained knowledge in build systems, integration testing and continuous integration with TravisCI. Then, they made some high level decisions that saw engineers leave. As the guy with the most knowledge of the cloud infrastructure, I was suddenly promoted to lead DevOps.

I was in way over my head, so I bought _[The DevOps Handbook](https://itrevolution.com/book/the-devops-handbook)_. That single book changed, I'm not exaggerating, everything. It all clicked. All the skills I had gathered over the years since graduation made sense. I wanted to learn all the best practices, the Toyota Manufacturing System, what an SRE is, and how to put it all in place. I was determined to try.

The following year was filled with putting forth the lessons in the book. I did my best to achieve the DevOps transformation, but it didn't happen. The failure still hurts and is lessons learned. The problem was management buy-in. They didn't see the need for the changes and just wanted me to continuously fight fires. So, I quit.

Finding a company with the management buy-in is how I found my current job. I interviewed them as much as they interviewed me. So far, they haven't lied to me. Even when they merged with another company, they allowed me to move the DevOps way up the chain so all our teams can practice those best practices. I feel lucky. I'm happy to be a SRE doing the best DevOps I can.

I know this isn't a helpful post about "How to Become a DevOps Engineer" but it's how I did it. I just kept learning. That's the real answer.
