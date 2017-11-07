---
layout: post
title:  "Finding orthonormal basis given vector!!"
date:   2017-11-07 21:22:36 +0100
categories: technical post
---

**Introduction**
==========================================================================================================================================================================================
Most of the times in graphics we do come across case when we are given some vectors and, we want to define co-ordinate system.  

For example, If we want our camera to follow the character in the scene  and, we have position of camera and position of the character.  

Now in order to do this we will need to find co-ordinate system for camera.  

In other words, we need to find orthonormal basis.   

Othnormal basis is nothing but set of vectors perpendicular to each other and unit in length.  

Given orthonormal basis by using linear combinations of the vectors defining basis, we can derive any vector in the given space.  

In our case we can treat co-ordinate system of camera as vector space of 3D vectors.  


**Algorithm**
==========================================================================================================================================================================================
For this purpose we can use process called as **Grahm schmidt** which is as follows  

In order to find orthonormal basis for camera we have two vectors   


+

  **v** - Vector got using position of character and position of camera.  
  
  **u** - Vector got using position of camera and origin of world co-ordinate system.  
  
  
  **Note : Camera is placed in scene with respect to some world co-ordinate system.**  
  

+

  Select vector **v** and normalize it.  
  
  **Note : We want out final vectors in orthonormal basis to be unit vectors. **  
  
  Select vector u and find projection of **u** onto **v**.   
  
  Please see below for formula :   

  ![Formula for projection]({{ site.url }}/assets/projutov.jpg){:class="img-responsive"}  
  

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~  
  For us the projection is simplified to :  *(u.v) * v *
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
  
  As we have normalized **u** so denominator in projection formula is nothing but **|u|** which is 1.  
  
  Please remembder projection of **u** onto **v** will give us new vector lets call it **projvu** which is in the direction of **v**.  
  

+

  Now using following equation we get new vector which is perpendicular to **v** and **projvu** . Lets call it **s**.  
  
  Please see image below :  
  

  ![Formula for projection]({{ site.url }}/assets/projdig.jpg){:class="img-responsive"}  
  

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
   s = u - projvu ; 
   s = normalize(s);
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~  
+

   Now we have **v** and **s** which are unit vectors and perpendicular to each other.   
   
   We just need one more unit and perpendicular vector to complete the orthonormal basis.  
   
   We can achieve it using following formula.  
   

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~  
   t = normalize(v X s)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

   Hence starting with a vector we got orthonormal basis defined by vectors **v,s,t** for camera.   
   