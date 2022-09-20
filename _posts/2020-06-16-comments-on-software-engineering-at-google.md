---
layout: post
title: Reviews on "Software Engineering at Google"
subtitle: My thoughts on Google's success in engineering
tags: [software, engineering]
comments: true
---

Recently I have read a great article called ["Software Engineering at Google"](https://arxiv.org/ftp/arxiv/papers/1702/1702.01715.pdf). There are many points inspiring me on software development principles. Actually these applies not only on software engineering, but also, I believe, on running a successful business.

### 1. "Most of Google’s code is stored in a single unified source-code repository, and is accessible to all software engineers at Google."

Contrary to the popular belief - source codes should not be the most important asset in the company. The intelligence shared across the company brings the value. In the long run, everyone should feel ownership for the whole company - you can understand the flow of every process in the company, and then contribute to them. The disclosure of source code to every developer does not give away the long term edge of Google but make them much more competitive than other players in the field. 

### 2. "If a bug is discovered, it’s common to track down the change that introduced it and to comment on the original code review thread to point out the mistake so that the original author and reviewers are aware of it."

I always believe when a bug happens in the software, it is not only the responsibility from the original author but also the reviewers (not only the approved but also the "no-show" ones). We should not only review why the original author miss to catch the bug, but also why every reviewer could not spot it out.

For most companies, RCA (Root cause analysis) is always used to follow up a critical issue in the production, so that the development team can explain the cause to the business holder or clients in full details. The main purpose of RCA is not to point out the scapegoat but to neutrally find out the loophole in the development cycle and propose an improvement to prevent the issue in the future.

When a mild issue occurred, the development team may take the time only to find the blame, fix it and close the ticket. Without writing down the mistake for the original author and the reviewers, when the issue reoccurred a few times, it may just become a "wise" within the team. It may not pass down to the new comers. It may not spot out in the rewrite and the same issue occurred in the rewritten software. It may not bring any awareness to the reviewers that they share the same responsibility to gatekeep the software.

RCA should not only be required on the critical issue, and should not only be an official document. It can be just a few lines of comment in the original PR.

### 3. Most software at Google gets rewritten every few years.

I especially like the full explanation in the article for the following points

- Software environment, technology, and user expectation around it change rapidly
- Rewriting code cuts away all the unnecessary accumulated complexity
- Encourage mobility of engineers between different projects

Sometimes you can easily spot out the two types of developers in the team. I name them as "enthusiasts" and "conservatives".

Enthusiasts: "This part is so dumb. And also another part is horrible. Let me rewrite everything"

Conservatives: "I don't like so much change. I'd reject every big changes in the pull requests"

When a pull request triggers a battle on these two types of developers, most of the times conservatives win the battle. Some may explain normally conservatives are the more senior developers, or in a higher hierarchy in the firm, than the enthusiasts. I'd admit it's one of the possibilities. At the same time, such event always happens on a simple business or technical change which raises the original author alert of a complex structure in the software. Both sides are correct - the structural problem does really need a change right now, but the resources allowed for the "simple" change is so limited to make such a great change.

So the firm absolutely requires a formal progress to rewrite the software among a certain period. With the official grant to the rewrite resources, the conservatives can just explain to the enthusiasts, "Let's do it in the rewrite coming in next quarter / month / week", while they can still focus on the tight deadline of the business change. The software structure can always be modified to adapt the changing business requirement. The enthusiasts can be allocated with resources to design and propose the rewrite change. Then they will both be happy.

### Conclusion

Finally, there are a lot of points I would like to address and highlight, e.g. "20% time on personal projects", "reliable and fast building system" and "five accepted programming languages with protocol buffers". I believe these are also very crucial to Google's success and great reminders to every development team. I'll see if I have time to address them in the next article.