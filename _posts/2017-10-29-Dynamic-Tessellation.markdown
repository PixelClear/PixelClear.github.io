---
layout: post
title:  "Dynamic LODs using adaptive Tessellation!!"
date:   2017-10-29 15:13:36 +0100
categories: technical post
---

{% include image name="Tess0.png" caption="Tessellation Overview" %}

**What was done before programmable pipeline ?**
=====================================================================================================================================

Dynamic level of details previous to programmable pipeline used to be implemented on CPU and then push modified vertices down the pipeline. There were two distinct disadvantages to this scheme,

1. we are bound by bus bandwidth when copying new generated vertices from CPU to GPU.As the level of detail increases and hence, more number of vertices needs to be copied.This will degrade performance of your application and make it bus bandwidth bound.

2. Whole work load of sub dividing surfaces is on CPU side hence, not taking much advantage of parallel nature of GPU. If there are very high level of details, work of CPU will be more. This will result in degrading performance and making your application CPU bound.

     Other solution to subdivision of surfaces on CPU is to keep multiple copies of Mesh with different LOD active in memory and depending on distance from eye position draw appropriate Mesh. Again this solution is not scalable and there is limited VRAM available and hence not best solution.

**Whats with Programmable Pipeline?**
=====================================================================================================================================

We now with programmable pipeline have tessellation shader stage.For given patch (patch is just bunch of vertices forming new primitive for tessellation) we can decide the outer and inner tessellation level on the fly and this will generate and produce new vertices on the fly. Following are advantages 

1. We can start with very low poly model. Hence when we sending vertices from CPU to GPU load is minimum.

2. We are generating vertices on fly so whole workload is on GPU.  As vertex shader runs per vertex ,Tessellation Control Shader is called per patch and this invocations could be in parallel . Also next stage of tessellation which is Fixed Function stage (Primitive Generator) will also be heavily parallel. Hence we are taking full advantages of GPU.

**How we can implement it?**
=====================================================================================================================================

There are couple of methods on how we can implement it.

1. View Distance Dependent Method

2. Sphere diameter in clip Space Method

In this article I have implemented first method. It is pretty straight forward to implement it.Following is the algorithm that describes those steps

1. For every edge of the primitive find distance of that edge and Eye Position.

2. Depending on this distance decide the Outer and inner tessellation levels.

**Problems with method I have implemented ?**
=====================================================================================================================================

For the primitives that share edges the outer tessellation level needs to be carefully generated.The outer tessellation level generated for edges shared by primitives must match else we will get cracks.For different orientations sometimes its very tricky to match this levels. Outer tessellation level decides how edge of the primitive will be divided while , inner tessellation level deals with how inner part of patch will be divided. Please see images below ( Credit to NVDIA )

![Tessellation Artifacts]({{site.url}}/posts/Images/Tess1.png){:class="img-responsive"}

![Tessellation Artifacts]({{site.url}}/posts/Images/Tess2.jpg){:class="img-responsive"}

**About Demo**
=====================================================================================================================================

 In start of the demo you will see I have loaded low poly version of model. After that, I enable dynamic LODs and you see more number of vertices generated as I move mode close and far from Eye Positions.
[Tessellation Video](https://youtu.be/ay3cHWwKi90)
