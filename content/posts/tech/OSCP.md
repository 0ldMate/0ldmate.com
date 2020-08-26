---
title: "My Attempts at OSCP"
date: "2020-07-15"
tags: ["redteam","certificates"]
---
There are so many different OSCP writeups out there, too many even. Why read mine? I failed the exam 3 times so maybe that will provide a slightly different perspective compared to many of the guides which pass on the first go. 

## Prior knowledge

Before I go through the course, I'd like to cover where my knowledge level was at. At the start of the year I was studying to do my CCNA so I spent a few hours each working day on it for about 6 months. I was ready to take the CCNA exam and was excited to become qualified in networking. My day job was a helpdesk/jnr sysadmin position in a medium size company which I had been at for over two years. I spent most of my time working on simple IT tasks like fixing printers and general computer issues. In my downtime at work I picked up a few extra things but had very little exposure to the offensive side other than at conferences/talks. After spending 6 months studying CCNA I self funded a trip to DEFCON, it was there I realised security is where I wanted to be, not networking. 

Once I returned home, I spent the next few months preparing for the OSCP. I didn't have the knowledge to go into it immediately so I started with two main resources:

- PentesterLab - this taught me a lot of the basics I needed to know and introduced me to a lot of different types of vulnerabilities
- HackTheBox - this is great to actively learn and practice exploiting machines

I can't recommend both of these tools enough when preparing for the OSCP. These can be used to create a great base of knowledge to build on. If you're unsure how to tackle HackTheBox, I would recommend looking at IPSec's videos and following along. In the span of 3 months I went from not knowing what to do after nmap scanning to almost knowing what to do after running nmap. I didn't feel ready for the OSCP when I signed up, but a friendly reminder that "OSCP is meant to be a learning experience" was the thing that pushed me into signing up.

## Lab Time

I began my labtime just before Christmas in December, I didn't feel ready for it, but the truth is you won't ever feel ready. I viewed OSCP as this huge task where I needed to be so prepared for that I could conquer it in one go, which is wrong. OSCP is a course, it is a learning experience and should be treated as such. I felt my basic understanding was good enough, I knew a little about reverse shells, scanning machines, what tools to use. So I jumped in with 90 days lab time.

### Day 1 - 60 OSCP 1.0

During my lab time there were a few major changes with the course, the main two were:

- Failed exam attempts had an extended cool-off period
- OSCP 2.0 came out

Due to the exam cool-off period I had my first exam attempt really early on so I could make sure I still had time to continue working in the lab environment. Whilst it was good to get a feel for the exam, I don't think this was the correct approach, I should have waited till I was more prepared. If you run out of lab time, Hack the Box is a great substitute.

OSCP 2.0 being released really threw a spanner in the works for me and after a week or two I purchased the upgrade with 30 days remaining. I hadn't finished my lab report on version 1.0 and having a short amount of time I wasn't going to spend the remaining time I had completing the new lab report which was now significantly longer.

At the end of my lab version 1 time I had owned about 10 machines (which included one of the big 4 and a network secret) with one or two machines where I only had user. I wasn't super happy with these results but I was excited to be working on updated machines.

### Day 61 - 90 New Version, New Me

Upgrading with 30 days left into a new environment was difficult and looking back on it I don't know if it was the right option. After upgrading you get a fresh updated document with loads more stuff in it which meant a lot more reading and lab excercises with a third of my lab time left. I read a lot of the new material whilst at work in my downtime and spent most of my time in the new lab environment which in many ways was similar, similar exploits just on a different operating system.

I didn't get to touch any of the new content like the Active Directory exploitation as it wasn't in the exam and I was just focused on practicing my methodology and hacking machines. I don't think I really benefitted upgrading so late into my lab time and as I was self funding this whole journey I definitely didn't want to fork out additional money for more lab time.

### Exam Attempt 1

As stated above I went into this exam attempt very under prepared and went in with the mindset of "It's my first attempt, i'm going to fail this, let's just see how it goes" which isn't a great mindset but it's something you often hear on OSCP forums. I took the exam 45 days into my lab time. I had practiced Buffer Overflow and had good notes on it so I was able to crush that machine within an hour or two easily. I then bounced around on a few boxes and ended up getting user on one additional machine and that was all. I did however gain a lot of information on the machines and kept my notes. For the machine I only got user on, there was a clear privesc option but I was unable to get it to work. Between attempts I was able to spin up that version on my local machine and attempt and troubleshoot the privesc.

What I learned:

- Notes are super important
- Practicing exploits offline is a good way to confirm if that's the correct path
- I needed to improve my enumeration skills
- Take longer breaks

### Exam Attempt 2

My second attempt I felt a lot more confident in my skills at enumerating machines and privilege escalation. My lab time had just ran out so I felt pretty fresh and ready to go. I was able to get the Buffer Overflow easily, and then proceeded to try other machines. I ended up getting administrator access on one additional box and user on another 2 machines. I felt super close to being able to privesc on those two other machines however I was unable to get them to work, until the last 30 minutes. Within the last half an hour of the exam, something clicked and I was able to get the priv esc to work but it triggered too late into the exam and I wasn't able to get the root flag, if I had gotten this I would have passed, I also believe I would have passed if I had completed the report. I then spent time afterwards doing the same thing I did after the previous attempt, I spun up the software and practiced the exact privesc.

What I learned:

- Previous exam notes are helpful sometimes
- Keep trying up until the last minute
- Knowing how to find resources/tools is super important

### Exam Attempt 3

At this point I was just over it, having to wait 2 months before I could attempt the exam again after missing out by such a small margin, I almost just didn't take another attempt. I had been preparing by tackling Hack the Box and reading walkthroughs. I felt as ready as I'd ever be going into this attempt. Starting out I missread my Buffer Overflow instructions and messed up, rather than owning the box in less than 2 hours I ended up spending 4 hours on it. After finally figuring out what I did wrong and getting root, I took a break and chilled out before going onto the next machine. I was lucky and had a few repeat machines so I was able to smash with breaks in between. After a few hours on the other new boxes I hadn't seen I ended up getting root on 4 machines with a metasploit usage and I had enough points to pass. I had made sure my notes were perfect with screenshots and kept working on the other machine till the very end. I had finished. 

What I learned:

- A Windows machine may be handy to have nearby
- Remember to not stress
- If the person who found the exploit has a blog, there will probably be useful info on it

I wrote up the report over the next day and submitted it and after several days of constantly checking my emails I got the email saying I passed. If I didn't and had to wait a further 3 months I doubt I'd be taking it again.

Overall, the experience was very unique and interesting. It really put me in the hacking mindset and made it really easy to fall in love with offensive security. Some times it does feel like it's more of a CTF than anything, but that's just part of the course. It exposed me to a lot of different exploits and methodologies that are a great base for knowledge to be then built on. A big part of my journey was possible because of how much I wanted to move jobs. I really disliked the work I was in and looked at the OSCP as the only way I could get into the line of work I wanted to get into. This made me put in a lot of hours after work and on weekends that lead me to achieving this certiifcate.

##Tips

Try working on this before work rather than after if you're struggling to get it done - I went into work at least an hour early everyday when training

Make sure you've got drive before going into this. For me, that drive came from I didn't like what I was doing for work and saw this amazing field I wanted to dive into.

Find a community to work with and bounce ideas off

Take good notes - I now use Notion, however previously I've used CherryTree

Keep your notes from previous exams - You never know when you'll see a repeat box

Hack the Box is a great substitute to the lab time if you run out
