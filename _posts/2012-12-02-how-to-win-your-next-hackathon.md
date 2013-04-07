---
layout: post
title: "How to win your next hackathon: 8 top tips."
tldr: "We won this Fall's AngelHack London and we'd like to share our experience with you."
published: true
---

On the weekend of 10-11th of November, we took part in the first ever <a href="http://angelhack.com" target="_blank">AngelHack London</a>. For those who haven't heard of it, AngelHack claims to be the world's biggest hackathon and holds events from San Francisco to Tel Aviv.

Judging from the amount of people participating in the London hackathon at Bloomberg's offices, their claim could quite possibly be true. Over 350 developers, designers and 'idea guys' attended with around 60 teams surviving until the end. It was a deeply impressive event from <a href="http://twitter.com/GGopman" target="_blank">Greg</a> and the rest of the AngelHack team.

### What we did
We designed, hacked, shipped and pitched a browser app called <a href="http://usepenny.com">Penny</a>. All this happened in about 22 hours.

We also 'won' the hackathon, along with two other great teams: <a href="http://www.bnkpositive.com/" target="_blank">BnkPositive</a> and <a href="http://testlio.com/" target="_blank">Testlio</a>.

At the time, we were exhausted and winning came as a complete shock. Looking back now at the array of 60 hacks presented that day, it's easy to think "damn, we got lucky".

Probably most of it *was* luck, but we think we did a few things right. Eight things, to be specific. We'd like to share them with you.

### 1. Prepare beforehand, have a plan

Have a clear plan of what you want to build before you arrive, if that's within the rules of the hackathon. If you're not allowed to plan the product, at least plan who will take what roles. Who's the front-end guy? Who's handling deployment?

We met and talked for about an hour or so the day before AngelHack. We didn't actually do much sketching or wireframing, but did talk about what was feasible and who would Penny's potential users be.

When it comes to planning, do whatever you think will help your team hit hard on the day. Just remember to play fair and don't break the rules.

### 2. Dream big, build small
Have a big idea. Tackle a big problem. Dream big dreams.

Now that you've done that, also accept that you *will not* be able to build everything you want to. You have to make sure that the portion of the product you've planned to build is feasible in 24 hours.

At a hackathon you need to build the subset of features that maximize your chances of someone understanding what you're trying to achieve and provide the most value to your end user.

For us, that meant abandoning slick user profile pages and not building some cool viral mechanics. Instead we focused on getting the main value proposition of Penny in place — alerting users when garments discount.

### 3. Be disciplined & committed

You're at a hackathon to have fun for sure, but you're also there to build. That means knowing when to put the nerf gun down and put your headphones on. Looking back, we had roughly a 'code-for-1-hour-then-check-in' cycle going. That helped us achieve small wins and plan ahead.

We know it's easy to get distracted and let time slip by, but you just need to think about how great it will feel to stand on stage at the end of the day and demo a product actually that works.

When the team at the table next to you start calling you the "hardcore guys", you're probably on the right track.

### 4. Use the clouds

We deployed a pretty decent infrastructure for Penny in just a day. We used Github pages to set up a marketing site for people to sign up for the beta, EC2 servers for the application and the crawlers, SQS queues for communication between the parts of the system, a hosted MongoDB powered by the great folks at <a href="http://mongohq.com" target="_blank">MongoHQ</a> and SES for sending out notification emails.

And we seriously suck at ops.

You've got to minimize the time spent on things other than your application and maintain developer happiness. If you're like us, going all sysadmin will probably not help that much.

### 5. Make it look & work good

It shouldn't come as a surprise that design counts. Even if your team is all developers who don't know their Arial from their Helvetica, it still counts. At AngelHack, we got the impression that design was a substantial differentiator for the judges.

If you have design skills in your team, take the time to craft an awesome interface. We were lucky in that regard and people were stopping by our desks to comment on how nice the app looked.

If you don't have the skills, try to find a designer on the day or use a framework and keep it simple. Either way, there are few excuses for not considering your hack's design.

### 6. Deploy

The first thing we did when we started hacking on Saturday was to make a <a href="http://fabfile.org" target="_blank">Fabric</a> script for deploying Penny. Pushing code to "production" (let's not kid ourselves, hackathon code is hardly production code) feels like progress and keeps you motivated during the deep hours of the night.

As an added benefit, if your project is accessible from the internets you shield yourself from laptop blunders during your pitch, since you can use any other available machine for demoing.

### 7. Have a demo, show it to people

Demo, demo, demo! You're being judged on your technical chops. Your idea might be brilliant, but you need to show that you can build it. Even just a frontend with ridiculous, fake numbers will get you a long way.

The most common question from the judges was "what did you actually build today?". If you can't show them something, your chances of winning are going to be impaired.

Also remember to demo to anyone you can grab throughout the day. We were lucky to know a few other people at the day who we could show our progress to. This not only feels good, but it can throw light on a problem that you've totally overlooked.

### 8. Pitch like a badass

At a hackathon like AngelHack, it's not enough to just build something, you've also got to pitch it. This can be super intimidating to a lot of people because it means standing up in front of a room full of very intelligent people and trying to sell your idea.

You have to consider a few things before you get up to pitch: do you know how you would extend your idea given time and money; do you know how it could make money; have you considered all the significant technical challenges (even if you haven't solved them); why are you the right people to be doing this.

Ultimately, if faced with tough questions, honest answers are the best. One of our judges turned out to be an expert in the space we'd positioned Penny. He asked some *very* good questions and at one point we even had to answer "yeah that's going to be a huge challenge".

But that's okay – when you think about it, it's those "huge challenges" that get us out of bed in the morning.

*Originally posted on <a href="http://usepenny.com/blog/how-to-win-your-next-hackathon/" target="_blank">http://usepenny.com/</a>*

