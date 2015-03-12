---
layout: post
title: "Tracker: How no one uses it in the same manner"
date: 2015-03-12 14:38:05 -0700
comments: true
categories: pivotal tracker
author: Kael Quet
---
After having used Tracker with six different teams over my time at three different companies, it's clear to me
that no one uses it in the same manner. No one seems to call it the same either: "Pivotal Tracker", "Pivotal", "Tracker".

## What have I seen ##

### Pointing ###

- Code first, point later. The lack of estimating prevents the PM from scheduling ahead of time.
Tracker doesn't let you start a story before pointing, as this flow may also lead to laziness.
- Lead dev pointing. Although better than the above, this can lead to a bias. The lead may not
have touched all areas of the code-base either, some things may be overlooked.
- Group pointing. What we do here. Majority wins, and everyone's experience is considered.

Example of Points:

- 0 copy/assets change
- 1 tests are required, sometimes assumed to take half a day
- 2 thought of complexity for a full day's worth
- 8 story is poorly written and should be split up

## What I want to see ##

### Pointing ###

Point the story independantly of what has already been pointed, even if similar to a previous story. The order at which stories are
completed is not always the same as displayed in the pointing session, nor can we assume that the person working on the story to be
pointed has full context on the previously pointed one.

### Separate projects for the different products ###

Currently our project iterations are planned automatically, which means Tracker will use our velocity to place stories from
the backlog into the current interation. We lose meaning of velocity due to the fact our different products/epics, will be at
different stages, and have different teams. If the products happen to have related stories, stories may be linked across projects
for reference. I suggest we create separate projects to isolate progress statistics, and enable
the use of automtically planned. Having separate projects will also help us with the following: release markers.

### Design to be placed on the epic ###

To avoid hunting which story has to do with a certain aspect of the UI, it would be great to
place the design in both the story related, as well as the epic itself. The design can be
attached to the epic with the particular story ID for more context.

### Release markers ###

These help the developers see what the timelines are given the stories in flight. Stories will show up as red if manually assigned to an iteration when the velocity calculated is insufficient. Again, these may be tough to implement if a project contains several products.

### Github's [service hook](http://pivotallabs.com/level-up-your-development-workflow-with-github-pivotal-tracker/) for tracker ###

Placing the story ID in the commit message will automatically add a comment on the story with a link to the github commit view.
If the the story ID is prepended with "#finished"/"#finishes", it will add the comment as well place the story in a delivered state;
however, this may not go well with our flow of waiting for honesty to pass before finishing a story.
