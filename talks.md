---
layout: page
title: Talks
---

## Why should you learn writing C extension?

### [PyCon Taiwan 2020](https://tw.pycon.org/2020/en-us/conference/talk/1159574667502027113/)

In the talk, I would like to give out the solutions to these myths by introducing modern approaches on writing C extension. Also I will try to convince the audience that, with the appropriate tools, migrating the pure Python code to C extension code in the performance critical components can be a great investment in the application.

[Slides](https://github.com/gavincyi/pycon-why-should-you-learn-writing-c-extension)

## I can't believe it's still here!

### [PyCon Japan 2020](https://pycon.jp/2020/en/timetable/?id=203309)

You may shout it out when you see a function should have been deprecated in 3 years ago. The function may be created by your ex-colleagues but no one manages it after his / her leave. It may hit the system now, or just silently stay in the great number of source files. You may still ponder the next move - remove it now, or let it stay there until the application retirement. The talk will dives into this scenario and purpose a systematic approach, auto-deprecator, to resolve the problem.

[Slides](https://www.slideshare.net/GavinYingInChan/i-cantbelieveitsstillhere)
[Video](https://www.youtube.com/watch?v=w4XEIGmH5Zc&ab_channel=PyConJP)

## Equip your performance toolbox - Cython v.s. Pybind11

### [PyCon Sweden 2019](https://www.pycon.se/2019/index.html#talks)
### [PyCon Ireland 2019](https://2019.pycon.ie/schedule/#session-47)
### [PyCon France 2019](https://www.pycon.fr/2019/en/talks/conference.html#equip%20your%20performance%20toolbox%20%E2%88%92%20cython%20vs.%20pybind11)

Developing Python applications is handy and rapid, but performance is always concerned, especially on the CPU bound components. We'll first go through the common tools and tricks which may surprise you on the performance improvement, and compare the two prevailing tools, Cython and Pybind11. Finally, their similarity and difference, in terms of implementation and performance, will be listed out so that attendees can thoroughly understand the criteria to choose the right library in their projects.

[Slides](https://github.com/gavincyi/pycon-presentation/blob/master/pycon-ie-2019-presentation-no-bg.pdf)
[Video](https://www.youtube.com/watch?v=ZRKjoUALmwk)