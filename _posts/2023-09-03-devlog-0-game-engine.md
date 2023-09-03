---
title: "Devlog #0: Creating a game engine from scratch, the begin"
date: 2023-09-03T00:00:00+00:00
toc: true
comments: true
toc_label: "Table of contents"
categories:
  - Blog
  - Technical
  - DevLog
  - GameEngine
tags:
  - blog
  - devlog
  - C/C++
  - game-engine
---

So, I'm finally kick starting my game engine **OhEngine**, [github link](https://github.com/eHonnef/OhEngine), I don't aim to be the next Unreal engine, it is just some hobby project for fun, that's why you should not expect too many updates on this blog, neither in the repository. The main idea is to provide a framework/system/platform for 2d and 3d games... So, yeah, it will be a looong project.

Disclaimer: This is my first time tackling this kind of project, I have the intention of developing and architect the system by my own, said that, the design decisions that I made are not rock solid and can change, also, I'm not an expert in the subject of game engines, so don't expect the "ultimate game engine design", I will just document what I have done, the challenges and the next steps. Of course, for that, I will check some materials and read the vast knowledge that the books and internet offers, I will try to link my source material in the chapter "references" on each devlog.

# The road so far

1.  **Devlog 0**
2.  [Devlog 1](2023-09-03-devlog-1-game-engine)

# Why?

Not for a particular reason, just a hobby project and mostly because I enjoyed the computer graphics topics from my university. Besides that, it can be a nice challenge, it will teach me a lot of new things, improve what I already know, my code quality and other benefits TBD.

# Initial development notes

It is a very big project, and a hobby project that probably you will be working alone mostly of the time, so you don't need to plan every detail from the beginning, start with some top-level planning:

- Which components do I need now?
- Without thinking about code: how they will interact with each other?
- Some basic use cases: how the final user will interact with my engine?
- What should I provide?

Then go and refine each layer, do one layer per time, refine this layer again, refine the components, now thinking about code, how the refined components will interact with each other?
Start with basic things, but modular that can be reused, hardly someone designs the perfect system out of the bat, so, do not lose motivation, just develop and see things happening and then make incremental improvements on what you already implemented.
You don't need to optimize things immediately, the important thing, for now, is to have a solid API, for example, initially you don't need a binary search, just create the search function and later, using the profiler data, you check where you are losing time and then optimize the module.

# License

The code will be under the [Mozilla Public License, v. 2.0](https://www.mozilla.org/en-US/MPL/2.0/). Refer to [FAQ](https://www.mozilla.org/en-US/MPL/2.0/FAQ/) on a quick overview on what you can and cannot do.

At first I used [LGPLv3](https://www.gnu.org/licenses/lgpl-3.0.html), then I changed to MPL 2.0, my reasoning are basically in the first 2 questions in the [MPL 2.0 FAQ](https://www.mozilla.org/en-US/MPL/2.0/FAQ/).

# The Roadmap

With a project of this size it is always a good idea to keep track on what to do and when to do things. I will not put the roadmap here because it can be changed, and it will, so I'm going to update it in the git [docs folder](https://github.com/eHonnef/OhEngine/blob/master/docs/Roadmap.md). But to not keep the article too dry, this is the roadmap I came so far:

These modules are refined and planned on how they will integrate with each other and with upper/lower layers in the engine stack:

- Implement the entry point;
- Debug systems with Logger and profiler;
- Implement the window api; (I will use [SFML](https://sfml-dev.org) at first)
- Implement the event system with the event manager and notifications;

These modules are not refined yet, they are just a square in the stack that will be implemented in the future, they still can be expanded into multiple classes or modules, but at this point, I already visualized how they will be integrated with the previous modules and in between the layers:

- Some STL wrappers for data structures;
    - It is useful because it will make easy in the moment we need to implement our own data structure or swap to a more performative one for that use case.
- Basic entity system;
- Very basic renderer;
- Math library;
    - Only a matrix class with the basic operators;
- Renderer;

Also, you can check the layers, components and how they interact with each other in the [LayerDiagram](https://github.com/eHonnef/OhEngine/blob/master/docs/LayerDiagram.drawio) file:

![diagram](https://github.com/eHonnef/OhEngine/blob/master/docs/LayerDiagram.svg)

Yes, it is not a proper UML diagram, the correct way would be a sequence diagram with the class diagram and so on.... Buuut, I just want to quickly visualize the layers and how the components interact with each other on a top level.

- White: means implemented;
- Red: means not implemented, but planned;
- Yellow: Third party, integrated in the code;
- Blue: Partially planned and partially implemented;

On this diagram, the top layer can use the lower layer, for example, the `CWindow` class uses the `Logging` system, and the lower layer cannot use the top layer nor has the knowledge about it, it can only notify via events or realizations. For example, the event system, the application class (`CApplication`) implements the event listener (`IEventsListener`) and the application has knowledge about the window class, but the window class has no idea that the application class exists, it only communicates with it via the event listener. In my opinion, this philosophy reduces the possibility of the code to become a spaghetti.
As you may observed, not all things are connected, for example, every module there uses the Logging system, but I choose not to draw it to avoid unnecessary cluttering, as this diagram can become very confusing. So this kind of thing is implicit, I will only draw what I find important to document.

# References

- [The Cherno - Game engine series](https://www.youtube.com/playlist?list=PLlrATfBNZ98dC-V-N3m0Go4deliWHPFwT)
- [Writing a Game Engine from Scratch series](https://www.gamedeveloper.com/programming/writing-a-game-engine-from-scratch---part-1-messaging)
- [Isetta Engine devlogs](https://isetta.io/blogs/week-0/)
- Foley, J. D., Van, F. D., Van Dam, A., Feiner, S. K., & Hughes, J. F. (1996). Computer Graphics: Principles and Practice. Addison-Wesley Professional.
- Gregory, J. (2018). Game Engine Architecture. A K PETERS.
- Eberly, David H. (2006). 3D Game Engine Design. A Practical Approach to Real-Time Computer Graphics. Morgan Kaufmann; 2. edition (3 Nov. 2006).
- Steinbruch, A. and Winterle, P. (2009) *Algebra Linear*. São Paulo (SP): Pearson Makron Books.
- Steinbruch, A. and Winterle, P. (2006) *Geometria Analitica*. São Paulo: McGraw-Hill.
