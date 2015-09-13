<!--
.. title: Don't use old OpenGL
.. slug: dont-use-old-opengl
.. date: 2012-11-20 01:24:33 UTC
.. tags:
.. category:
.. link:
.. description:
.. type: text
-->

... or at least think twice before you do.

*audience: Computer graphics students and OpenGL enthusiasts*

<!--more-->

The distinction
---------------

I'll start with explaining the difference between "old" and "new" OpenGL as I see it.

By "old" I mean OpenGL version 1.x through 2.x.
By "modern" I mean the core profile of versions 3.x and higher.

Identify them easily by characteristic functions:

- Old OpenGL uses functions like `glBegin`, `glVertex3f`, `glLightfv`, `glPushMatrix`
- Modern OpenGL uses `glDrawArrays`, `glVertexAttribPointer`, `glUniform`, `glCreateShader`.

OpenGL has evolved along with the architecture of GPUs: newer versions are progressively more programmable (shader-oriented).

There's a big ideological gap between "old" and "modern" that you should understand:

- Old OpenGL provides lots of built-in high-level functionality (matrices, materials, lights, vector specification) that looks easy to set up and use directly in your application.
- Modern OpenGL got rid of all that, and instead talks in terms of various low-level objects like memory buffers and shaders that lets you *program* your GPU to render whatever you imagine.

Some prefer to learn older OpenGL because useful, understandable features are available out-of-the-box, so you can see the results quickly. Modern OpenGL programming interface has a steeper learning curve, because there's much more you need to code from scratch to see your first "hello, rotating cube". That said, this interface is more comfortable to use in the long shot, because of its flexibility.

My recommendations
------------------

- If you want to **learn graphics programming**, learn modern OpenGL. You'll gain more insight into how graphics cards work.
- If you just want to **write a visualisation or a game**, don't use OpenGL directly, use a 3D graphics engine like Irrlicht. OpenGL (be it new or old) is still rather low-level and you'll need to pay attention to lots of technical details to use it effectively.
- If you're experienced and you're writing a graphics engine **with maximum compatibility in mind** - then indeed you have a good reason to use an older OpenGL version. 
