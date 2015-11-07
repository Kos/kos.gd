<!--
.. title: Introducing Glory
.. slug: introducing-glory
.. date: 2013-03-28 16:07:40 UTC
.. tags: projects
.. category: dev
.. link:
.. description:
.. type: text
-->

Glory is the code name of my prototype IDE for developing OpenGL-powered graphical effects. After half a year of work, it has entered a functional alpha stage (and awarded me the&nbsp;MSc.)

<a href="/images/screen1.png"><img src="/images/screen1.png" alt="screen1" width="980" /></a>

<!--more-->

My main inspiration were excellent, yet a bit vintage shader design tools such as RenderMonkey or FX Composer, both last updated around 2008. A lot has changed by then. I didn't try to make a copy, though: during the design, I made it a priority to avoid inventing my own abstractions on top of OpenGL. In other words, I tried to expose OpenGL in an user-friendly, but accurate way. The idea is to avoid limiting the user's possibilities to some subset that I've predicted.

As a matter of fact, I find this distinction (managing a domain's complexity vs. hiding it away) very blogworthy and applicable to many areas. Expect me to rant about it sometime soon.

There are two target groups for the program:

- CG students and enthusiasts, willing to learn OpenGL or experiment with it,
- Professional CG artists who want a comfy platform to prototype shaders and complete effects.

The alpha is functional but rather bare-bones. It has reached the point where a good study together with target users is necessary to decide where to put the most effort. Merrily following the feature creep is tempting, but how to make the thing user-friendly for newbies? How to allow professionals integrate it into their workflow? Hard to answer that without user studies.

I have yet to decide about Glory's future and invent a business model for it; this kind of challenge is new to me and I'm all excited. Chances are it will be easily available, though :-)

Glory is written using Python and Qt (PySide). During the development, inspired by the fun we had with WPF at [Blue Brick](http://blue-brick.com/) last year, I've conjured am MVVM microframework for Python that has succeeded to save me from some Logic <-> UI boilerplate. I believe it might prove useful outside of my app, and partially even outside Qt. I'll clean things up and share it on GitHub soon.

