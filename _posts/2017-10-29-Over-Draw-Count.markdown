---
layout: post
title:  "Over Draw Count is your enemy!!"
date:   2017-10-29 15:13:36 +0100
categories: technical post
---

**What is overdraw count ?**
=====================================================================================================================================
Mostly on embedded hardware the major concern for performance drop could be overdraw. Basically one pixel on screen is shaded multiple times by the GPU due to nature of geometry or scene we are drawing and this is called as overdraw. There are many tools to visualize overdraw count.

**Details about overdraw?**
=====================================================================================================================================
When we draw some vertices those vertices will be transformed to clip space then to window coordinates. Rasterizer then maps this coordinates to pixels/fragments.Then for pixels/fragments GPU calls pixel shader. There could be cases when we are drawing multiple instance of geometry and blending them. So, this will do drawing on same pixel multiple times.This will lead to overdraw and could degrade performance.

**Strategies to avoid overdraw?**
======================================================================================================================================
 1._Consider Frustum culling_ - Do frustum culling on CPU so that objects out of cameras field of view will not be rendered.

 2._Sort objects based on z_ - Draw objects from front to back this way for later objects z test will fail and the fragment wont be written.

 3._Enable back face culling_ - Using this we can avoid rendering back faces that are looking towards camera. 

If you observe point 2, we are rendering in exactly reverse order for blending.We are rendering from back to front. We need to do this because blending happens after z test. If for any fragment fails z test then though it is at back we should still consider it as blending is on but, that fragment will be completely ignored giving artifacts.Hence we need to maintain order from back to front. Due to this when blending is enabled we get more overdraw count.

**Why we need Per Pixel Mutex?**
=======================================================================================================================================
By nature GPU is parallel so, shading of pixels can be done in parallel. So there are many instance of pixel shader running at a time. This instances may be shading same pixel and hence accessing same pixels.This may lead to some synchronization issues.This may create some unwanted effects. In this application I am maintaining overdraw count in image buffer initialized to 0. The operations I do are in following order.

a) Read i pixels count from image buffer (which will be zero for first time)

b) Add 1 to value of counter read in step 1

c) Store new value of counter in ith position pixel in image buffer

As I told you multiple instance of pixel shader could be working on same pixel this may lead to corruption of counter variable.As these steps of algorithm are not atomic. I could have used inbuilt function **imageAtomicAdd()**. I wanted to show how we can implement per pixel mutex so, I have not used inbuilt function **imageAtomicAdd()**.

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
#version 430


layout(binding = 0,r32ui) uniform uimage2D overdraw_count;
layout(binding = 1,r32ui) uniform uimage2D image_lock;

void mutex_lock(ivec2 pos)
{
  uint lock_available;
 do
 {
   lock_available = imageAtomicCompSwap(image_lock, pos, 0, 1);

 }while( lock_available == 0);
}

void mutex_unlock(ivec2 pos)
{
  imageStore(image_lock, pos, uvec4(0));
}

out vec4 color;
void main()                                                                                 
{                  
     mutex_lock(ivec2(gl_FragCoord.xy));           
     uint count = imageLoad(overdraw_count, ivec2(gl_FragCoord.xy)).x;
	 count = count+1;
	 imageStore(overdraw_count, ivec2(gl_FragCoord.xy), uvec4(count));
	 mutex_unlock(ivec2(gl_FragCoord.xy));                                                
}

Fragment_Shader.fs

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

**About Demo**
======================================================================================================================================
In demo video you can see we are rendering many teapots and blending is on.So pixels with more intensity shows there overdraw count is high.

Note : On android you can see this overdraw count in debug GPU options.

[Demo Of OverDraw Count](https://youtu.be/Ko8ctJQeewY)

