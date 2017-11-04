---
layout: post
title:  "Faster Fragement Shaders Using Tessellation Shaders!!"
date:   2017-11-03 21:22:36 +0100
categories: technical post
---

![]({{ site.url }}/assets/TeaPot1.jpg){:class="img-responsive"}

**Faster Fragement Shaders Using Tessellation Shaders**
=======================================================================================================================================
If there are very complex operations in fragment shader then, we do see some performance drop. As the execution frequency of fragment shader is per fragment hence, its very high.This is one of the techniques discussed in paper by Wang et al (See reference) and by Erick (See references) to simplify and auto tune fragment shader. Generally lighting calculation can be done in vertex shader or fragment shader. Depending on what quality we want we generally decide place of calculations.Doing lighting calculations in fragment shader will generate great results. So its trade-off between quality and performance.

**Vertex Shader Lighting**
=======================================================================================================================================
![Vertex Lighting]({{ site.url }}/assets/TeaPot2.jpg){:class="img-responsive"}

**Fragment Shader Lighting**
=======================================================================================================================================
![Fragment Lighting]({{ site.url }}/assets/TeaPot3.jpg){:class="img-responsive"}

**Why Vertex Shader Lighting results are not that good?**
=======================================================================================================================================
We in this example are showing specular lighting. This is high frequency signal.When we do lighting in vertex shader, we do it per-vertex based.When calculated color passed through pipeline the rasterizer will do work for us and interpolate the result to fragments.If you see above image you can observe image is not so pleasing. We are not getting enough samples for this high frequency signals as number of vertices are very small and so result is not very nice.

**What is problem?**
=======================================================================================================================================
So as result of above issue we move our lighting calculations to fragment shader but then we get hit in performance.Please see results of lighting calculation in fragment shader.

**What is solution?**
========================================================================================================================================
If we move our lighting calculations to tessellation evaluation shader we can approximate this lighting calculations. Our initial problem was small number of vertices so, with help of innerTessellationLevels and outerTessellationLevel we generate more number of vertices. Hence indirectly we are creating more number of samples and approximating the lighting calculations.In this case rasterizer do interpolate color between vertices but as number of vertices are more and interpolation happens over small domain. Please images below.
![Tessellation Levels]({{ site.url }}/assets/Teapot4.jpg){:class="img-responsive"}

**Tessellation Level 1**
========================================================================================================================================
![Lighting with TL1]({{ site.url }}/assets/TeaPot5.jpg){:class="img-responsive"}

Above image is taken with tessellation level 1. You can see we are getting results somewhat similar to vertex shader. Following images are taken with tessellation level 5 and 10 . You can see after tessellation level 5 we get results similar to fragment shader.

**Tessellation Level 5**
=======================================================================================================================================
![Lighting with TL5]({{ site.url }}/assets/TeaPot6.jpg){:class="img-responsive"}

**Tessellation level 10**
=======================================================================================================================================
![Lighting with TL10]({{ site.url }}/assets/TeaPot7.jpg){:class="img-responsive"}

P.S: Though this specular lighting is not much performance extensive, I am explaining idea here which can be applied in appropriate instances.For example we can use this approximations with Procedural Texture Shader. This shader uses Perlin Noise to calculate procedural tetexture in real time.

**References :**
=======================================================================================================================================
[Ref 1](http://www.cad.zju.edu.cn/home/bao/pub/36.pdf)

[Ref 2](https://erkaman.github.io/posts/tess_opt.html)
