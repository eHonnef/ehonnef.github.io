---
title: "Devlog #1: Creating a game engine from scratch, the begin"
date: 2023-09-23T21:00:00+00:00
toc: true
comments: true
toc_label: "Table of contents"
categories:
  - Technical
  - DevLog
  - GameEngine
tags:
  - devlog
  - C/C++
  - game-engine
---

On this devlog I will describe the challenges, choices and the implementation of the Windowing and event systems, unit testings (?) and why I implemented wrappers for the STL.

Wow, 2 blog posts in a row, what happened?
It just made more sense to split into 2 posts, one about the idea, road map, whys and some basic concepts, and this one that dive more into technical details on what was implemented.

Git commit for this post: https://github.com/eHonnef/OhEngine/commit/aa5342b7c69e1864051f83e0f1889373b377afe3

# The road so far

1.  [Devlog 0](2023-09-03-devlog-0-game-engine)
2.  **Devlog 1**

# Why not start with rendering stuff?

Mostly materials that I read had the same warning: "Don't start with the renderer". (you can check the material in the previous devlog) Now... why not? Here are the reasons from what I read, of course, this is not written in the rock, so take with a grain of salt:

- The event system is one of the most important, messaging around the code stack .....
- The renderer is one of the biggest system, therefore, it will be a colossal task, and it can be demotivating.
- Having other systems already ready will make the renderer programming easier, because you won't need to implement the support systems half way just to make that part of the renderer work. Think, for example, in the event system, if you still don't have one, you probably will need to write one while writing the renderer, and that can lead to mistakes or spaghetti. Writing the support systems beforehand will make your life easier in the long run.
- Have some basic entity object, so you can feed it to the renderer, this way the renderer will already "know" the kind of object it should handle.
- Debugging and testing a renderer in isolation can be difficult. Many issues might only become apparent when other engine components are integrated. Starting with a well-rounded support system allows for more comprehensive testing and debugging from the beginning. Because you know that those support systems are good to go.
- The learning curve, if you're developing a game engine as a learning experience, beginning with the renderer might be overwhelming. Learning about rendering techniques, shaders, and graphics APIs can be complex on its own. Starting with a simpler component like entity management or input handling can provide a gentler learning curve.

While the renderer is a crucial component of a game engine, starting with it might not be the most efficient or effective approach. It's generally advisable to begin with a solid foundation that includes the core architecture and systems of the engine and gradually integrate and optimize the renderer as part of a more comprehensive development process. This way, you can ensure that all components work harmoniously to create a functional and efficient game engine.

# Quick topics

Topics to go through very quickly, could be some bullet point.

## Custom containers... why?

Yes, I could definitely used directly the containers from the C++ standard library. My reasoning for the wrappers? If, in the future, I want to change the type of the container, for example, from [std::vector](https://en.cppreference.com/w/cpp/container/vector) to [std::list](https://en.cppreference.com/w/cpp/container/list), then I don't have to refactor a lot of code, I just change the internals of the container's class and I'm good to go. In shortly, I want to avoid API changes.

## Unit testings

There are a few for now, mostly of them are for checking for stupid mistakes, for example, if the list wrapper is behaving as it should, if the input event translation unit is mapping to the correct values, and so on.

Definitely I will implement more testings in the future, there are things that I need to learn how to automatically test, for example, the window itself, how to check each event, the dispatcher, etc etc.

# Window

Just for fun, I gave these bullet points to ChatGPT and asked to make a blog post (yeah, I'm a little bit tired after writing the Events sections). You can consider these bullets points as a TL;DR, and you can skip to the [Events](#events) section, that one I wrote myself :)

- At first I wanted to use the [GTK library](https://gtk.org/) since I had some experience with that, but I couldn't control the window rendering, that's because GTK has its own "main loop", [here is a link that explains a little bit about it](https://stackoverflow.com/a/47156941), and it updates via event signaling, that would be ok, but I want to control the moment that the draws in the window is updated.
- That's why I have chosen the [SFML](/app/joplin-desktop/resources/app/sfml-dev.org/) library for this, one of the reasons is the nice C++ api... give more reasons to use SFML.
- I have implemented a facade for the SFML window
- I used the [pointer to implementation](https://en.cppreference.com/w/cpp/language/pimpl) pattern for that, so I can completely hide the SFML library

## ChatGPT's text about my decisions on the window system

When embarking on my recent development project, I initially gravitated towards the GTK library. After all, it was a familiar tool in my programming arsenal, and I had some experience with it. However, as I ventured deeper into the project, I soon realized that I needed more control over window rendering - a level of control that GTK's inherent structure simply couldn't provide.

Here's the story of how I transitioned from GTK to the SFML library and discovered a world of newfound possibilities:

### **The GTK Roadblock: Lack of Control**

One of the major roadblocks I encountered with GTK was its reliance on its own "main loop" and its event-driven rendering updates. While this approach works well for many applications, it proved limiting for my specific needs. I craved the ability to dictate precisely when the drawings in my window would be updated. GTK's event signaling just didn't cut it for me.

### **The SFML Revelation: More Than Just a C++ API**

Enter SFML, the Simple and Fast Multimedia Library. My decision to switch to SFML was influenced by several key factors, with the exceptional C++ API being just one of them:

1.  **Elegant C++ API**: SFML offers a C++ API that's not only elegant but also a joy to work with. Its well-designed syntax and straightforward functionality make it a fantastic choice for developers comfortable with C++.
    
2.  **Total Control Over Rendering**: Unlike GTK, SFML empowers developers with a high degree of control over the rendering process. With SFML, you can decide precisely when and how your window's contents are updated, allowing for optimization and fine-tuning to achieve your desired results.
    
3.  **Facade for Simplified Usage**: In my SFML implementation, I created a convenient facade for the SFML window. This abstraction layer simplified the interface, making it more user-friendly. It effectively shielded the complexity of the SFML library from the rest of my codebase.
    
4.  **Pointer to Implementation (PImpl) Pattern**: To ensure a clean separation of concerns and to further encapsulate the SFML library, I opted for the Pointer to Implementation (PImpl) pattern. This smart architectural choice allowed me to completely hide the inner workings of the SFML library, exposing only a well-defined interface to the rest of my application.
    

### **In Conclusion**

My transition from GTK to SFML was a pivotal moment in my development journey. The combination of a refined C++ API, fine-tuned control over rendering, and the ease of abstraction through a facade and PImpl pattern not only resolved my initial challenge but also elevated the overall quality of my project.

If you've ever found yourself grappling with similar constraints or value the features I've highlighted, I wholeheartedly recommend giving SFML a try. It might just become your go-to tool for graphics programming, offering you the control and flexibility you need to bring your creative vision to life.

# Events

My first idea was to use the observer pattern, [quick reference](https://en.wikipedia.org/wiki/Observer_pattern), in which an object, named the subject, maintains a list of its dependents, called observers, and notifies them automatically of any state changes, usually by calling one of their methods. This design, at first, looked interesting for an "Event Manager" where events like key presses, mouse movements, and other interactions trigger specific actions. The Application class would play the role of the observer, and various events (`OnKeyPressed`, `OnMouseMove`, etc.) would be implemented into it, while the Event Manager would have a list of observers to notify. However, a few problems emerged:

1.  Redundant Code: The events (OnKeyPressed, OnMouseMove, etc.) should be implemented in all observers regardless;
    1.  I'm aware that I could leave empty virtual functions to avoid that like: `virtual OnKeyPressed(args) {}`
2.  API Changes: As the application evolves, new arguments might need to be added to event callbacks. This could lead to cascading changes across all the observer classes, causing potential disruptions and maintenance difficulties.
    1.  That could also be true for my current implementation, but since each event is encapsulated inside a class, I have more room to play around without changing too much the code in the future.
3.  Encapsulation: While encapsulating events within separate classes might provide some advantages, it could complicate the code base and hinder future flexibility.

## So, what have I implemented?

I used the idea from "The Cherno" video where he implements the [visitor design pattern](https://en.wikipedia.org/wiki/Visitor_pattern), this approach simplifies the event management process by introducing a single event handler function called "OnEvent," which dynamically dispatches events to their corresponding subscribers. The benefits:

1.  Centralized Event Handling: Instead of implementing individual functions for each event, the application features a single "OnEvent" function that receives and processes events based on their type.
    
2.  Subscribed Events: Developers can subscribe to specific events by registering their callback functions with the dispatcher. This decouples event handling from the event source, promoting a more organized and efficient code structure.
    
3.  Flexibility and Maintenance: With the dispatcher pattern, adding new events or modifying existing ones becomes simpler. Changes are localized to the dispatcher, minimizing the impact on other parts of the code base.

Here is the sequence diagram overview of the system, for now I simplified the `CApplication` main loop, we are only interested in the event system, also abstracted the "SfmlWindow" and "Layers" class. About the flow, first our Application class will handle the event that it registered, for example, the window close event, then we propagate (if applicable) the event to the layers class.
The idea of the layers class is that if one of the Layers handled the event, then we immediately return. Think the layer class as multiple windows stacked to each other, if I click the top one, I don't want this click/event to be propagated to the windows below.

![diagram](https://github.com/eHonnef/OhEngine/blob/master/docs/SequenceDiagrams/EventSystem.svg)

SFML allows me to poll the events when I want, so... that's quite convenient, specially if I want to put a queue of events in the future, the downside is that the user could feel it, the famous "input lag", but that is something that future me will handle.
A little bit about the code flow, starting in the `CApplication` loop, we update the window, in the window update we check if there is any event that should be handled (we poll the window's event queue, all of it) then we update the internal `EventState` if the event is one of these: Key press/release, mouse movement or mouse click, after that, we propagate the event to our listener, the `CApplication` class, where it handles to itself via the `CEventDispatcher` and propagates to the future Layers class where it should also handle with the `CEventDispatcher`.

Quick mention about the input translator, if we want to use another window system, for example, [GLFW](https://www.glfw.org/), it will, for sure, have another set of keyboard key enumerators, and that will result in an API change. So to avoid that, we have our own input enumerator for Keyboard, Mouse, Joystick, etc, and what we need to do is map from the GLFW input to our input, it is a lot of manual labor, but that's the price to pay. For now, I copied the SFML enumerators and that will be our input enumerator, it is pretty nice [check out](https://github.com/SFML/SFML/blob/78973e4a06dd5ace2185329ad165d391cc3c01a8/include/SFML/Window/Keyboard.hpp#L51C9-L51C9), so since it is the same, we don't need to make a map, we only cast one enumerator to another.

About the Event dispatcher, the cool part is that we are not obliged to implement all the different handlers for each event and we only have one call with the same parameter: `OnEvent(CEvent& event)` , so no need to call `listener.OnMouseMove()`, `listener.OnKeyPress()`... we just call `listener.OnEvent()`. That, of course, comes with one downside (that I know so far), we need to cast the event to the desired type in run time, and that can pile up if we have thousands of events being generated per second. But, again, that's something that we need to check down the road, when we have something more complex to render.
The dispatcher works by checking the received event type (function `GetEventType`), and the template parameter that we give when we call `Dispatch` by using the function `GetStaticType` and if it is the same, then we can call the given function to handle the event, if this handler function returns true, that means that the event was handled by that function and we can stop propagating the event down the layers.

And it worth mentioning that, up until now, the event system is blocking, that means that the next frame only will be drawn after we finish handling all the events from `pollEvent()`.

### Event polling system

Sometimes the user wants to control when to check the events instead of being interrupted by a call from the event system, or the user doesn't want to keep track of the pressed keys, or the modification keys state (e.g.: if the Ctrl key is pressed), and so on. So, instead of doing that by himself, let us handle this for him, so he can ask the engine for the information about the mouse position, pressed mouse and keyboard keys and the modification keys state.

This is handle inside the `EventState` class.

# Wrapping up / TL;DR

- Starting with the renderer might not be the best idea in game engine development, since it is one of the biggest component. It's recommended to build a solid foundation first.
- Implementation of custom containers because they give you flexibility. If you decide to change container types later on, it won't require a massive code overhaul.
- Unit testing is a bit limited right now, but it's important for catching silly mistakes and ensuring the engine's stability as we go along.
- Choose to use SFML for the GUI system. SFML's C++ API is much friendlier, and we've hidden the library from the user with pointer to implementation.
- We used the dispatcher pattern for the event system instead of the observer pattern, tt's a more organized way to handle events and makes future changes easier.
- Our input translator helps us stay flexible. If we ever switch to a different window system, it won't break everything because we've got our own input enums.

