<!--
.. title: Python likes DLLs: Starting out with OpenGL and GLFW
.. slug: python-likes-dlls
.. date: 2012-10-11 01:21:45 UTC
.. tags: python, opengl, polyglot
.. category: dev
.. link:
.. description:
.. type: text
-->

This article shows how to load and use dynamic libraries in Python, taking OpenGL and GLFW as the example.
While some code is Windows-specific, the guide relates to other platforms too.

<!--more-->

Loading GLFW DLL
----------------

The first thing we're going to do is load the GLFW DLL using CTypes. Here's how:

	import ctypes
	glfw = ctypes.cdll.GLFW

This will work if GLFW.dll is found by the OS. Otherwise supply the full path:

	glfw = ctypes.cdll.LoadLibrary('path/to/GLFW.dll')

I've used the `cdll` loader (which also works on Linux for .so libraries), but for some DLLs on Windows `windll` will be required:

> "cdll loads libraries which export functions using the standard `cdecl` calling convention, while windll libraries call functions using the `stdcall` calling convention."

Having loaded the DLL, you can call all functions from it. Works like a charm!

	glfw.glfwInit()

	GLFW_WINDOW = 0x00010001
	glfw.glfwOpenWindow(640, 480, 8, 8, 8, 8, 24, 8, GLFW_WINDOW)

	while 1:
		glfw.glfwPollEvents()
		glfw.glfwSwapBuffers()

The constants like `GLFW_WINDOW` are #defined in the header file (`glfw.h`) so keep the header file open for reference (or extract all of them automatically).

The sad part is that ctypes is unable to find the function signatures, so if you pass wrong argument number or types, then you're likely to have some silent bugs. (On Windows, `ctypes` is nice enough to examine the stack and tell you that the arguments take too many or too few bytes than the function expects; this may also happen if you use an invalid calling convention.) However, you can work that around by filling the signatures yourself!

Read up on `ctypes` in the [Python documentation](http://docs.python.org/library/ctypes.html). Here's a simple example:

	glfw.glfwSleep.argtypes = [ctypes.c_double]

	glfw.glfwSetTime.argtypes = [ctypes.c_double]
	glfw.glfwSetTime.restype = None

	glfw.glfwGetTime.argtypes = []
	glfw.glfwGetTime.restype = ctypes.c_double

Now you can call these functions happily and `ctypes` will do its best to convert between C types and Python types according to the signature you've entered.


More juice: GLFW + OpenGL
-------------------------


Let's mix some OpenGL in here now! On Windows, OpenGL 1.1 functions are present in `opengl32.dll`, let's load it:

	gl = ctypes.windll.LoadLibrary('opengl32')
	# or just...
	gl = ctypes.windll.opengl32

That's it! *(On linux, it's probably just 'GL' IIRC.)*

	gl.glClearColor.argtypes = [ctypes.c_float]*4
	gl.glClearColor(0.2, 0.3, 0.4, 1.0)
	gl.glClear(0x00004000)
	glfw.glfwSwapBuffers()

If you're on Windows, you'll only be able to access OpenGL 1.1 like this. Accessing newer functions is easy too, but needs a bit more effort:

	>>> hasattr(gl, 'glCreateShader')
	False
	>>> hasattr(gl, 'wglGetProcAddress')
	True

`wglGetProcAddress` is a part of Windows API that lets you load extension functions or core functions from OpenGL > 1.1. It allows you to ask for an OpenGL function by name and get a function pointer. (Remember that it needs an existing OpenGL context to work.)

	>>> gl.wglGetProcAddress('glCreateProgram')
	1466130944
	>>> gl.wglGetProcAddress('glDebugMessageControlARB')
	1466183040

*(Note: Different platforms have different entry points for that purpose, like `glXGetProcAddress`; GLFW provides a cross-platform wrapper called `glfwGetProcAddress` that you can use.)*

It's a bit tricky to obtain a Python callable function from a numeric function pointer. We have to first create a "function type" object, then use it to create the function object itself:

	>>> functype = ctypes.WINFUNCTYPE(ctypes.c_int)   # CFUNCTYPE for cdecl, WINFUNCTYPE for stdcall
	>>> functype
	<class 'ctypes.WinFunctionType'>
	>>> glCreateProgram = functype(gl.wglGetProcAddress('glCreateProgram'))
	>>> glCreateProgram
	<WinFunctionType object at 0x02AD1558>
	>>> glCreateProgram()
	1

You should repeat that for every OpenGL function or extension function that you need. Note that C or C++ programs often do exactly the same process using extension loader libraries (like gl3w, GLEW or GLee).

Indeed a tedious task to do by hand! One thing you could do is to load a C loader library just like we did with GLFW before, but there's a nice alternative available.

Enter PyOpenGL
--------------

Look how far we've gone without using any external Python libraries! However this one seems worth a shot. Install it using your favourite package manager (like pip).

	>>> import OpenGL.GL as gl
	>>> gl.glClear(gl.GL_COLOR_BUFFER_BIT)  # Yay, the constants are here too
	>>> gl.glCreateProgram()                # As well as modern OpenGL functions
	2L

These functions can operate on the same OpenGL context, but have the advantage that they are prepared for nice interaction with Python: type signatures are simplified so that you won't need to deal with ctypes manually (it can get tedious when C pointers get involved), OpenGL errors and GLSL compilation errors are thrown as exceptions (unless disabled), etc. There's also a logging facility.

There's a [documentation section](http://pyopengl.sourceforge.net/documentation/opengl_diffs.html) dedicated to differences between C OpenGL API and PyOpenGL. Also, expect a performance hit compared to C code.

Conclusion
----------

The setup I've described is lightweight, quick to set up, portable (GLFW is mature and cross-platform) and comfy to use - hey, it's Python after all! It may prove useful for prototyping or for production when sub-C performance is acceptable.
