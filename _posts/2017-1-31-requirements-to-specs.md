---
layout: post
title: From Requirements and Design To Specifications
---
## A summary of Jackson and Zave

People ask what the greatest benefit has been as a result of having done both software QA and web development. They assume I might release fewer bugs than other developers; that it's perfect on the first attempt. Maybe my unit tests just write themselves magically. True as these may be (they aren't), the biggest benefit of having had my feet in both worlds is my ability to consider the code I write from the perspective of someone who doesn't know how the code was written. It's my ability to think like a user. In practice, this boils down to the difference between considering two different software development tools: user requirements and technical specifications.

Knowing what separates a list of requirements from a list of specifications requires that we put on a few different hats. Requirements can be said to describe relationships in some environment; an environment being a browser running a website, for example. Specifications then would describe the way some program that we've built manipulates this environment. Specifications are going to be pretty comprehensive. Indeed, we can think of a program as the same as a list of specifications, only now all the items in the list will be executable. In this article I'll describe the process of deriving specifications from requirements. This document borrows heavily from the paper ["Deriving Specifications from Requirements: an Example"](http://dl.acm.org/citation.cfm?id=55864) by Michael Jackson and Pamela Zave.

In order to define this process, we're going to need to lay some groundwork and define some of its parts. I mentioned _environments_ in the preceding paragraph. What is an environment?

I, being a web developer, typically build programs which have to function as part of some web application running in a browser on either a desktop machine or mobile device. These things - an existing web application, a web browser, some hardware to run it all - are parts of an environment. In addition to these technological components of the environment, there are a couple of other very important components: the user, and time.

Into this environment I want to build and install some new program, or as Jackson and Zave call it, a _machine_. The job of a machine is to act upon the environment in such a way that the states in the list of requirements all come to exist. The machine is a wish fulfiller.

This is all well and good, but how then does a programmer take a list of requirements and use it to build some machine? Let's take a look at an example. Suppose we have a web application that already exists. Our users have asked us to install a new button in the application. The button in this case will, when pushed, cause a modal popup to appear on the page. Printed in this modal will be the number of times the button has been pushed. The modal will have a little red "X" button that, when clicked, will close the modal. Thus ends our requirements for a new button.

This is fantastic. I love buttons.

Note how every component in the requirements describes some condition in the environment, or some relationship betweens things in the environment. There are user-initiated events. The descriptions refer to things that will be apparent whether or not our user knows how the machine is accomplishing them.

The job of knowing how a machine will cause these relationships to exist is ours. Let's start by listing all the apparent events that can be performed using this machine:

Event | Event Name
--- | ---
User clicks button | ClickButton
App adds 1 to # of times button has been clicked | IterateClicks
App loads modal with iterated number of button clicks | LoadModal
User clicks "X" to close modal | ClickClose
App closes modal | CloseModal

Nice. These are in an order where each event follows chronologically after the event which immdiately preceded it. Notice that the outcomes of these events may or may not result in some of the observable phenomena described in the requirements. The event IterateClicks, strictly speaking, doesn't produce anything observable in the environment. Sure, it's a necessary step for the following events to function as required; it just isn't shared between the machine and the environment. All the rest of the events in the list produce shared phenomena. This separation of interestes is a key difference between user requirements and program specifications. A user might say "I don't care how you do it, just make sure the number in that modal is correct every time." Aye aye, intrepid user.

Whether a phenomenon is shared between program and environment is one thing, but it doesn't describe who or what controls that phenomenon. For example, a button click is shared because the user does it and the program receives it, but only the user can control whether the button has been clicked. Likewise, the modal popping up happens in response to the events that precede it, but the enforcement of that logic can only be controlled by the program, and not by the user. Any phenomenon can only be spontaneously controlled by either something in the environment (a user clicks a thing), or by something within the machine (a preceding event demands the next action be performed).

Let's update our list of events with their shared status and their controlling actors:

Event | Event Name | Shared? | Controller
--- | --- | --- | ---
User clicks button | ClickButton | Yes | User
App adds 1 to # of times button has been clicked | IterateClicks | No | Program
App loads modal | LoadModal | Yes | Program
User clicks "X" to close modal | ClickClose | Yes | User
App closes modal | CloseModal | Yes | Program

Now we can see the benefit of thinking like a user in addition to thinking like a programmer. Good requirements explain every phenomenon a user would expect to witness in the given environment, but they wouldn't have anything to say about program-controlled, unshared phenomena. In fact, a list of specifications is a superset of requirements. It'll contain all the relationships between phenomena in the environment (all the stuff a user sees and does on a page), and it'll also describe the behavior of the machine within the environment (how the new code interfaces with the environment).

At a high level, the task of deriving specifications from some given requirements is to list all the events kicked off from the environment, and insert all the behind-the-scenes events that the program will have to perform in order to make each next phenomenon a reality.

There are gotchas. What we've described is one path through this new button experience; what some might call the "happy path." What happens when there's a modal already loaded and the user clicks that button again? To specify the outcomes for unexpected events like this, we need to to talk about the state of our environment.

A state is a collection of phenomena described in user requirements. What kind of states can the environment be in with regards to our new button-modal functionality? This can be determined by examining the environment-controlled events in our list of requirements. We have two: ClickButton and ClickClose. We can determine future state from an event by hopping down the list of all the program-controlled events that are preceded by the given environment-controlled event. For example, a user performing ClickButton causes the program to perform IterateClicks, and then LoadModal. After that, time passes until the user or environment performs some other action upon our program. This is one state of our program. Number has been iterated, modal has been loaded. There is one other state, triggered by a user's ClickClose action: modal has been closed.  These two states fulfill our simple requirements.

In a perfect world, we can assume that the requirements are comprehensive. The "open" button and the "close" button do what they do, and that's all that they should do. Let's dwell in this perfect world a while.

The requirements give us the two states of this program. Our needing to lock the "open" button while the modal is loaded need not be in the requirements, but it does need to be in our specifications. Remember that requirements are a complete description of phenomena in the environment - all states are described. Specifications, being a superset, are the states described in requirements, plus the additional means to get there. Locking the "open" button while the modal is loaded should therefore be added to our specifications in order to fulfill the requirements. 

Event | Event Name | Shared? | Controller
--- | --- | --- | ---
User clicks button | ClickButton | No | User
App adds 1 to # of times button has been clicked | IterateClicks | No | Program
App locks "open" button | LockButton | Yes | Program
App loads modal | LoadModal | Yes | Program
User clicks "X" to close modal | ClickClose | No | User
App unlocks "open" button | UnlockButton | Yes | Program
App closes modal | CloseModal | Yes | Program

I think we've explored the outcome of every environment-provided event in our requirements. But we're not done yet! In most cases the environment where we're installing our new program is going to provide plenty of other events that might affect our state. What if the browser is minimized? What if JavaScript is disabled? What if everything needs to be controlled by a keyboard instead of a mouse or touchscreen? Outcomes branching from existing events available through the environment will need to be specified in order for our machine to diligently create the states described in our requirements. Exploring these paths often leads to necessary changes in the requirements themselves.

Think like a user. A user needs the machine that you've built to deliver some expected results. A user does not necessarily need to see the gears grinding in order to deliver those results. I, in my role as gear-grinder, find it helpful to remind myself every now and then that the machine is not the outcome of my efforts. The user's environment is.
