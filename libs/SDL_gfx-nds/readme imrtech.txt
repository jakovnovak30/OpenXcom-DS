

Ported lib needed to compile Xpired dc port Mostly makefile changes only.Compils with precompiled  lib for kos1.1.9 only.

Info
SDL graphics drawing primitives and other support functions
The SDL_gfx library evolved out of the SDL_gfxPrimitives code which provided basic drawing routines such as lines, circles or polygons and SDL_rotozoom which implemented a interpolating rotozoomer for SDL surfaces.
The current components of the SDL_gfx library are: 

Graphic Primitives (SDL_gfxPrimitves.h) 
Rotozoomer (SDL_rotozoom.h) 
Framerate control (SDL_framerate.h) 
MMX image filters (SDL_imageFilter.h) 
The library is backwards compatible to the above mentioned code. Its is written in plain C and can be used in C++ code.


http://www.ferzkopp.net/~aschiffler/Software/SDL_gfx-2.0/ homepage and info


(c) A. Schiffler, 1999-2003, licensed under the LGPL 