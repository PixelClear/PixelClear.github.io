---
layout: post
title:  "Most Interesting Bug I Encountered on GPU!!"
date:   2020-07-03 15:13:36 +0100
categories: technical post
---

**What was happening ?**
=====================================================================================================================================

Application was crashing very randomly.

**What happens when multiple application is executing ?**
=====================================================================================================================================

Multiple applications need GPU. OS has module that manages the access to GPU. 
When ever application with high priority comes and requests GPU access the currently executing application will be
preempted and context is rolled. New context is setup and GPU is given to new
application.

**What is GPU context ?**
=====================================================================================================================================

To fulfil draw request from application, GPU needs bunch of state.
Application/driver maintain per-draw state and change it on GPU.
There are multiple blocks on GPU that does specific tasks.
For example, you can set some state* so that the blender will blend values correctly.

These states are stored in registers and those registers are maintained in banked RAM on GPU. 
Each block that needs particular state also maintains copy of state.So multiple copy of the state registers are present.

**What is context roll ?**
=====================================================================================================================================

Driver communicates with block on GPU called as CP(Command Processor).
This CP block communicates state with multiple other blocks present on GPU.
Context roll is when CP asks particular block to start working on new values because, some state is changed by application.
CP does not work on its own. User mode device driver (UMD) issues multiple commands to CP.
Driver copies those values for new state in free bank and issues command to CP stating to look for new state.
Driver basically takes new state and prepares something called as PM4 packet. PM4 packet is form of sending commands to GPU from UMD.
If you are interested in further exploring it please see : [PM4 Ref](https://raw.githubusercontent.com/GPUOpen-Drivers/pal/28a98ba3e787278dad958afd2cadbdabf28bacfc/src/core/hw/gfxip/gfx9/chip/gfx9_plus_merged_pm4_it_opcodes.h)
CP parses this PM4 packet and searches for IT_SET_CONTEXT_REG and IT_LOAD_CONTEXT_REG and further asks other blocks to update it.
Then CP further asks other blocks update their copies.

**What was happening ?**
=====================================================================================================================================

The driver was failing to set the proper value for one of the registers in case of context roll and that was messing the drawing state and application use to crash.
As the context roll could occur randomly the application used to crash randomly.