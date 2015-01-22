---
layout: post
title: "Effective Engineering Onboarding"
date: 2015-01-22 15:00:00 -0800
comments: true
categories:
author: Andrew Purcell <andrew.purcell@grandroundshealth.com>
---

When I started at Grand Rounds in November, I was excited to learn how a new organization works. There's a commonly stated heuristic that it usually takes new hires three to six months to start being valuable at a new organization. After a month here, I felt like I was really contributing to my projects, and that got me thinking about the onboarding process.

The goal of onboarding is to put a new hire on a track to being a productive and happy team member. While the learning curve will differ from company to company, I think there are some common elements that your onboarding process should contain.

#### 1. Designated Mentor ####

Entering a new organization is difficult. People who have worked at a company for a long time have usually developed a certain way that they do things. They have opinions  and practices that may differ from yours. In some cases, this is a feature. You hire more senior people _because_ of the differing opinions and experience they bring. In other cases, new hires just don't know how your organization works. Since the goal of onboarding is to get the new person assimilated, you want to find these differences and resolve them as quickly and painlessly as possible.

A designated mentor solves a lot of problems for a new hire. Generally, they will assign work, review that work, and answer questions. They'll communicate cultural values and expectations.

Having a designated mentor is expensive, because some new hires may need a lot of hand-holding, and that's time that the mentor needs to spend in place of getting their own work done. There is a case for having a new person ask around until someone has the bandwidth to answer their questions. This might even cause them to look harder for answers before asking someone else, since it's tedious to explain your problem over and over to people who are busy. To this end, ask yourself: "how quickly do I want this person to get up to speed?" and "is the information they need actually in a place they can find it in a reasonable amount of time?"

#### 2. Good Written Instructions ####

One effective way to help your new hires and save mentor time is by writing up 'Getting Started' documents in advance. Have a task template on your wiki containing a list of first week tasks, and instruct your new hires to clone it and check it off as they go, asking for help when they need it.

At a smaller company, this step might not be as important. The documentation might take too long to write, or things might change so rapidly that the upkeep is cost-prohibitive. You might not be able to save much mentor-time by doing this step.

At a larger company, this step can be an incredible time-saver. There's often a lot of documentation around process, tools, and culture already written somewhere. Collect all these links on one page, and indicate what's important to read and what's extra. New folks, or even folks moving from other teams, can learn a lot by exploring a wiki.

#### 3. Technical Overview (via presentation or documents) ####

This goes along with good written instructions. It's important for an engineer to know where their work will fit into the larger picture of technology at your company. You may already have good documentation about the different components and applications that your business uses. If not, write some, or have another engineer present it.

In the case of a company with better existing resources for discovering how things are put together, give the new hire a task to research and present on the architecture. This will help them learn it, as well as give other engineers a refresher and opportunity to share some more subtle points if a new hire gets it wrong.


#### 4. Thorough Code Reviews ####

Most companies have style guidelines (but whether or not they're written down and people follow them is another question). If you don't, maybe you've gotten lucky so far, or you work by yourself.

**Extremely thorough** code reviews are one of the best ways to share your coding standards with a new developer. Sometimes, code reviews can hurt a developer's ego, so it is important to set the right tone:

> _This code doesn't meet our standards, here's how we can make it better._

and not:

> _Your code is awful and you are stupid._

It is better to point out nits (naming conventions, formatting, etc) in the first few code reviews than to wait until they want to merge a large feature story. Once coding standards are known, it's much easier to follow them. I used to hate applying `@CheckNotNull` to all of my parameters when writing Java in a certain codebase, but it did make [Findbugs](http://findbugs.sourceforge.net) more useful, and I did get used to it.

One of the biggest annoyances with stylistic code reviews is the feeling that your reviewer is impeding you from doing real work because of trivialities. To that end, get the "trivialities" out of the way early!

#### 5. Clear Tasks to Work On ####

Onboarding is about defining a good progression to get from starting to contributing. The best way to progress is to succeed at increasingly complex tasks. The progression might take two steps, or it might take ten, but it needs to end with the recipient feeling confident and ready to tackle new problems.

When a new developer joins a project, they need a good starting point. Most often, this is a set of small bug fixes or enhancement requests. Small tasks allow them to write a little bit of code, write a test, and commit it. They can watch how it goes through the build system, code review process, and then makes it into production. Going through the code lifecycle builds confidence: they've seen how it works, and they have an expectation for what should happen in the future.

On the other hand, sometimes new developers get handed vague projects. I have often seen this with interns. They struggle to interpret the requirements, and if they don't have a mentor who is invested, end up building (or not building) some half baked solution that solves nobody's problem. They try to give important sounding status reports in meetings but they're not getting anything done. It is a very demoralizing place to be.

To be clear, vague and underscoped requirements are a general problem in software development, **but** they are also incredibly damaging during onboarding. Instead of making a new person feel like a productive member of the team, they feel new and useless until the hobbling project is eventually killed, and they get to work on something else.

Of course there are exceptions. Sometimes you hire someone specifically for the purpose of tackling a vague project that you don't know what to do with.

### In Summary

A lot of the points above go beyond onboarding, and reflect positive elements of a healthy engineering organization. Some of them may not be applicable based on the size of your organization. A startup might have you contributing in a week, while at a larger organization it might take you three months.

Think about your onboarding process and if you tick these boxes. Did I miss something? [Let me know.](mailto:andrew.purcell@grandroundshealth.com)
