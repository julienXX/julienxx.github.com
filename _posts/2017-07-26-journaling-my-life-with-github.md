---
title:  "Journaling my life at work using Github"
date:   2017-07-25 17:18:00
description: "My daily routine using issues."
tags: ["thoughtleadering", "practice", "self-quantification", "github"]
---

Following this [post](https://lobste.rs/s/j9dkuv/do_you_journal_about_your_workday_why_what) on lobste.rs and the interest that some manifested after I mentioned it on twitter, I wanted to write about something I'm doing for about 6 months now which is journaling my days at work.

It started while chatting with my friend [Flavien](https://twitter.com/fcogez) who was asking how he could have a TODO list with reccuring tasks.

Half-jockingly I said use Github issues with a [template](https://github.com/blog/2111-issue-and-pull-request-templates). Thinking more about it I thought it could be a great idea and I should try this in practice.

## My daily routine

Regarding my daily tasks, I've mostly been a low-tech person with pen and paper crossing out what I had done. But this process is quite inefficient regarding reccuring tasks. Also keeping up with what was left to do from the other days is not really optimal. Tried some TODO/GTD apps too but it didn't catch.

These days I try to keep a journal of what I do at work using Github issues. I chose Github since I use it everyday so I figured this would be a great place to store this "me vs. work" stuff.

Every morning I open a new issue in a private repo with the date as the title. The repository is not public, perhaps one day it will. I've got mixed feelings with opening it, it's my private life but journaling in the open could be an interesting experiment too.

There is an ISSUE_TEMPLATE file in the repository. It contains my daily routine tasks and a placeholder for what I intend to do today and another one for tasks I did that were not planned.

The template looks like this:

```
Today here are the tasks I'd like to complete:

- [ ] Check Slack for urgencies
- [ ] Check X status
- [ ] Check New Relic
- [ ] Check Sentry
- [ ] Check Y

I also did:

- [ ]
```

So every morning with my coffee I make a quick standup with myself and it ends up with something like:

```
Today here are the tasks I'd like to complete:

- [ ] Check Slack for urgencies
- [ ] Check X status
- [ ] Check New Relic
- [ ] Check Sentry
- [ ] Check Y
- [ ] Project 42
    - [ ] a sub task
    - [ ] another subtask
    - [ ] and another subtask
- [ ] Fix bug #66

I also did:

- [ ]
```

I then add a label with my current mood and if I'm working remotely or not.

During the day I check every task I completed and eventually add un-planned tasks. At the end of the day I'm left with:

```
Today here are the tasks I'd like to complete:

- [x] Check Slack for urgencies
- [x] Check X status
- [x] Check New Relic
- [x] Check Sentry
- [x] Check Y
- [ ] Project 42
    - [x] a sub task
    - [x] another subtask
    - [ ] and another subtask
- [x] Fix bug #66

I also did:

- [x] Free-up space on Jenkins
- [x] Some Open source stuff
- [x] Fixed a bug for a customer
```

When the day is over it's time for a bit of reflection over it and I'll tag the issue with some labels to rate it. So far I have 12 of them:

<img src="/assets/images/labels.png" alt="Github labels list" style="width: 200px;"/>

The `productivity` labels are purely subjective, it's just how I feel at the end of the day not how much I accomplished quantitatively. The mood I set in the morning can be adjusted if I feel like it was not the right one too.

I usually close the issue but leave the tab open so that I can copy paste the unfinished tasks for the next issue.

## What's next?

The closed issues list is pretty nice with all the labels and the progress bar

<img src="/assets/images/issues.png" alt="Github issues list" style="height: 200px;"/>

but I haven't found a real use for them yet except a sense of completion (or not) at the end of the day.

Since I have an API with pre-formatted data at hand using Github, I'm thinking about making a visualisation interface with some graphs and try to get some insights. It would be a nice side-project to play with some new front-end tools too (Purescript I'm looking at you).

The idea would be not only to have a fancy TODO list but also to try to self-quantitize my life as a worker. Trying to extract tendencies from this data would be great. For example if it's been 20 days I've been setting my `Mood` to `Bad` and my `Productivity` to `Bad` then perhaps it's time for a deeper reflection and a big change, this kind of stuff. It could also help for the 1-to-1s with my manager perhaps.

 I also thought about some elisp or a script I could write to automate my process a bit but I'm not sure it would really add value. My issue routine is a moment of reflection, I'm not sure I want to speed it up right now.

That's it for my daily routine. If you have thoughts, comments or ideas of improvement don't hesitate to open an issue [here](https://github.com/julienXX/julienxx.github.com/issues).
