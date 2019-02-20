<!--
.. title: How not to fix burnout
.. slug: how-not-to-fix-burnout
.. date: 2019-02-20 13:21:09 UTC
.. tags:
.. category: dev
.. link:
.. description:
.. type: text
-->

January is a good time for retrospectives. As I'm writing this, my team is planning a "best failure of 2018" contest, so I thought it's a good opportunity to write down my personal entry.

Here's the story of that time when I left a team to fight burnout, and lost something more important in the process.

<!--more-->

## Starting point

It was my third year in the same product team responsible for a medium-size webapp. My time in the team was very good overall. Of course there were interesting projects and trivial projects; there's been lots of team member rotation and focus shifts, there's been unexpected focus pivots, lots of things.

Generally I didn't mind staying in one team for a long time, because both the work itself and my responsibility was evolving. I never suffered from doing the same thing for a long time.

Still, burnout eventually crawled in and I decided I needed a change.

## Reasons for burnout

The main problem I wanted to get away from was lack of focus and overload of responsibilities I collected over time. Chronologically:

Responsibility #1: Product development. The bread and butter of a product team. This includes both implementation planning and actual coding.

Responsibility #2: Tech support (second line) - mostly investigaton of system problem reports + development of small, non-planned product changes (such as customisations or security improvements) in order to make particular key customers happy.

Responsibility #3: Leadership responsibilities, including 1-1 meetings with developers who reported to me, and also recruitment and interviewing.

Each hat was interesting in its own way, but they all competed for the same time, which constantly made me feel I'm not doing well enough in each. My ability to get satisfaction from my work suffered because of this.

## Actions taken

First stage: I stepped away from product development. I continued to help with requirements and planning work, but I didn't actively code in that area any more. My reason for this move was that I was often away from the team, doing unrelated things, and wanted to avoid blocking the critical path with my absences. To compensate, I ensured that remaining 2 developers can fully focus on product without being distracted by tech support drop-ins.

After some time I made this distinction more official by leaving the product team, taking away the tech support responsibility with me. Next, I joined the support team (which previously didn't include any developers) with the intention to increase its coverage, so that items that would previously escalate to product teams could get handled in the support team directly. In addition, I hoped to start expanding the support team to cover some customer success duties, so that some customer requests that would previously become product team drop-ins would have an official path where they are prioritised and implemented.

## Outcomes

There's a mix, I'll start with the positives.

### The burnout issue has been addressed successfully.

For some time I got to be my own product owner and choose my own priorities. This was very refreshing, as I had opporutnity to more thorughly address long-lasting pain points. For example, investigating problem reports of test-takers was historically difficult. I finally found time to address the problem on a lower level by improving log collection from the test-taking interface and coming up with some log aggregating scripts. (Log aggregation was an unsolved problem for us at the time. Brr...)

Working as a solo developer also meant that I didn't block my team while taking care of non-code responsibilities. This alone decimated the amount of stress I experienced at work.

### Team flexibility has been increased

While doing support, I had enough flexibility to contribute part-time to different teams as needed. Once I joined one of the product teams as a "consultant" for the duration of a particular project (great success!).

### Product team became more focused

This point is controversial and I'm on the edge whether it was a net plus for the company.

When leaving the product team, I took with myself the responsibility of responding to tech-support queries. On one end, this allowed the team to focus more thoroughly on its primary goal: product development. Less context switches = good, right? Still, I heard some mixed reactions about this consequence.

For one, part-time support responsibility used to allow new hires to get a more thorough understanding of the system as a whole. At the price of more context switching, the team would grow more code ownership and wider system expertise. Since these traits bring fruit in long-term and come at the cost of very immediate product development speed, it's hard to grade if it's worth it.

Personally, I prefer to work in a team that aims to build product competence wider than the next few user stories. It's more in line with sustainable product development. So from this perspective, I don't think that removing the responsibility for support made the team better, even if it made it more productive in short term.

### I had much less impact on company success

It makes sense to have a self-sufficient support team. Still, support work is inherently reactive - working on various issues as they appear. I ended up often taking care of issues which were urgent, but not all that important. My role wasn't defined in terms of a particular problem to solve, and didn't even have a way of telling if I'm successful or not. This is not a healthy place to be in.

A potential improvement would be to start thinking about growing the Support team into a Customer Success team and take a more active approach to making clients satisfied. This idea was in the air at the time, but I didn't feel confident enough to drive it myself, which is a pity. Trying and potentially failing would have been better.

### My personal tech development suffered

This came to me as a surprise. I would have expected that working solo would bring more space for technical growth, but it didn't materialize due to other factors. Most importantly, the mode of work I chose didn't favour starting large projects with opportunity for continuous focused work and larger scale of challenge.

## Mitigation

I ended up regretting this decision. It fixed the problem at hand - burnout - but at the cost of losing touch with my own development goals and the company direction. At the time the decision felt like problem-solving, but in retrospect it looks more like problem evasion.

I have some ideas what I should have done instead, and the best one is: _go on a quest!_

Eventually, the quest found me. The company got a new CTO and slightly pivoted in product direction, which also affected the team structure. I ended up moving to a 2-person team responsible for a more experimental project focused around Kubernetes. This was more than enough for me to clear my mind of burnout while learning something new in the area I wanted to improve the most (dev-ops). This project was interesting on its own, but that's a story for another day.

Quests are a great way to do a proper mind-reset and escape a plateau of professional development. I got an opportunity to do one at work, but there are other options, such as a several week long deep-dive into an open-source project of your choosing (bonus points if you can convince your company to offer patronage, instead of taking vacations).

Regualar vacations work too! Next time I observe burnout, I'll take a month off to focus on woodworking or electronics or music. I'll make sure to document how it worked...

