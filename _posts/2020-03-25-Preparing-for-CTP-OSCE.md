---
title:  "Preparing for CTP/OSCE?"
date:   2020-03-25
tags: [posts]
excerpt: "Recently, I've been receiving a lot of questions surrounding the CTP course, and my goal is to answer as many of those questions/tips within this blog."
---
Introduction
---
<img src="{{ site.url }}{{ site.baseurl }}/images/offsec-student-certified-emblem-rgb-osce.png" alt="">

I want to note that this isn't meant to be a 'course review' necessarily, but more of the approach I took in preparing for the CTP course and exam to become OSCE certified.

Pre-Registration Knowledge
---
Before I decided that I wanted to go for the CTP/OSCE, here's where I stood:
- Held the [OSCP](https://www.offensive-security.com/pwk-oscp/)/[OSWP](https://www.offensive-security.com/wifu-oswp/) certifications (Molding the 'Try Harder' mantra)
- Went through [Corelan's - Exploit Writing Tutorials (Parts 1-3b)](https://www.corelan.be/index.php/articles/)
- Completed the SLAEx86 course offered at [PentesterAcademy](https://www.pentesteracademy.com/course?id=3)

Pre-Registration Challenge
---
<img src="{{ site.url }}{{ site.baseurl }}/images/fc4-me.png" alt="">

Unlike the PWK and WiFu courses, you cannot just sign-up for the CTP course. There's a [pre-registration challenge](http://www.fc4.me/) that needs to be completed. 

This challenge is meant to test your knowledge to ensure that you're ready for the course. 

So please, don't try to cheat yourself by trying to Google search the solution that others have posted. 

Do your due diligence.

Course Preparation
---
After completing the pre-registration challenge and setting my lab access start date, I attended [@pandatrax](https://twitter.com/pandatrax)'s two-day course called 'Beginner Exploit Writing: Zero-to-Hero Series' that was taught in Charlotte, NC at an ISSA conference. 

This course went over exploit writing on x86 architecture for Linux and Windows. 

The topics covered were vanilla overwrites, Structured Exception Handlers (SEH), egghunters, and bypassing Data Execution Prevention (DEP). 

Overall, it was a great refresher on what I was taught in the Corelan tutorials.

Lab Environment
---
Once I gained access to the labs, I dedicated all my personal time to studying. 

I was easily spending 5hrs/day on weekdays and 8-10hrs/day on the weekends studying. 

I understand that many individuals cannot dedicate this much time due to having jobs/families/other obligations. 

For me, this was feasible at the time because I was single and didn't have any other obligations outside of work.

The lab consists of eight modules covering four different topics (two modules per topic), when excluding 'The Networking Angle'.

Topics Covered
---
1. The Web Application Angle
2. The Backdooring Angle
3. The 0-Day Angle
4. Advanced Exploitation Techniques 

Lab Approach
---
1. Read through the module in the pdf
2. Watch the correlating videos for the module, while following along in the pdf
3. Complete the module, while following along with the module videos
4. Complete the module without module videos or pdf

I didn't move on to the next module until I understood the attack process, and why that process/path was taken. 

So in other words.. my browser was filled with dozens of bookmarks referencing commands, terms, topics, etc.

Once I felt comfortable that I had a solid understanding of the module, I moved on and repeated the same four-step process. 

After each module I completed, I went back to the previous module(s) and tested my knowledge as sort of a refresher. 

If I couldn't successfully complete a previous module, I made a note of where I fell short. 

At the end of each week, if I had any notes written down, I'd go back and do further research on those topics that caused me to fall short.

Exam: 1st Attempt
---
The exam is made up of four machines that you'll have to compromise by various means, totaling up to 90 points. You'll need to accrue 70 points to be eligible to pass. I say 'eligible' because you need to submit a report disclosing the steps you took to compromise those machines. These steps need to meet the requirements given to you in the exam details section that you receive when starting exam. Failure to follow the rules/requirements will thwart your chances of passing.

After spending several months in the labs, I decided I'd have a go at the exam. I felt comfortable enough with all the topics covered in the course to give it a shot.

I signed-up to take the exam two weeks out, giving myself some time to prepare a little more and to make sure my schedule would be cleared.

After preparing for the next two weeks reviewing some course material, and going through some vulnerable applications found on Exploit-DB (such as VulnServer), I sat for the exam.

Exam Overview
---
1. I failed; Was eligible for 45 points out of the 90 points available
2. I still had some learning to do; which isn't a bad thing!

_Note: Although I didn't pass, it was still an amazing experience. Offensive Security provides the right amount of material within the course for you to succeed. It's up you to build on that foundation they're offering, to be successful._

Post-Exam
---
After concluding with my first exam attempt, I knew exactly what topics/areas I needed to focus on and improve. I spent the next two weeks researching and understanding the topics of which I was lacking in.

After those two weeks of studying, I felt comfortable enough to reschedule for my second exam attempt.

Unfortunately, I had to schedule my exam for December 22nd, which is obviously right before Christmas (sorry family). I had set a goal for myself to become OSCE certified before the end of the year. So I had no choice but to schedule for last week right before the new year! :)

Exam: 2nd Attempt
---
Long story made short, I passed on my second attempt. I submitted my exam report and was eligible for all 90 points. I received the 'We are happy to inform you' email on Christmas Eve. That email was one of the best Christmas presents I could have asked for!

<img src="{{ site.url }}{{ site.baseurl }}/images/osce-cert.png" alt="">

Exam Tips
---
Here's a few tips I'd recommend to those heading into their first exam attempt. The following are what I followed for my second exam attempt, after learning what not to do based on the first attempt:
1. Get a good night's sleep - I took ZzzQuil to help myself fall asleep. If I didn't, I would have been overthinking/analyzing the exam and not of been able to fall asleep.
2. Schedule for an early time slot - I took an 08:00 time slot for my second attempt. This way my mind was fresh from a good night's sleep, and not be fatigued from being up doing other activities.
3. Eat - I made sure to eat good (healthy) the day before and the day of the exam. (Bang energy drinks are my go-to source for caffeine)
4. Breaks - There's nothing wrong with taking breaks. I took breaks around regular breakfast/lunch/dinner times, and took a few 30 minute breaks just to step away and get some exercise in.
5. Go slow, stay calm - You're given 48 hours to complete the exam. You don't need to rush.. the last thing you want to do is rush and make a mistake somewhere along the line without catching it.

Resources
---
Below are a few resources that I found quite helpful in my journey. I won't list everything, but enough to get you going in the right direction. Remember that researching is part of the learning process. 

1. [Jumping with Bad Characters](https://buffered.io/posts/jumping-with-bad-chars/)
2. [CTP/OSCE Study Guide](https://tulpa-security.com/2017/07/18/288/amp)
3. [PCMan FTP Server 2.0.7](https://www.exploit-db.com/exploits/37731)
4. [Back to the Basics](https://www.securitysift.com/windows-exploit-development-part-1-basics/)

I'll leave the rest up to you.. and best of luck on your journey to becoming OSCE certified! 