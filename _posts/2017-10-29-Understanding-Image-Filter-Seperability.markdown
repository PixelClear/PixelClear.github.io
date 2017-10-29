---
layout: post
title:  "Understanding Image Filter Seperability!!"
date:   2017-10-29 15:13:36 +0100
categories: technical post
---

Normally in image processing we use 2D filters (which is 2D function) which is applied to image (which is also 2D function) to get some interesting effects like blur,edge detection,sharpening and so on.The result image is produces as sum of products of this two 2D functions.

Now we can simplify this 2D filter by doing something know as filter separability. A two-dimensional filter **"S"** is said to be separable if if it can be represented as convolution of two one-dimensional filters **"V"** and **"H"** such that **S = V * H** .

Suppose we have two-dimensional Gaussian distribution with a standard deviation of 0.85.

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
h =
    0.0626    0.1250    0.0626
    0.1250    0.2497    0.1250
    0.0626    0.1250    0.0626
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If you observe above matrix and perform some reduced row echelon operation on this matrix you will see this is matrix with rank 1. By rank one we mean that there is only one column with pivot element or in other words there is only one column and one row which are linearly independent.Matrices with rank 1 has special property that this matrices can be represented in **ROW vector*COLUMN vector** form.For example,
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
A =
    1   4   5
    2   8   10
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

A is Rank 1 Matrix.
So Lets break it up.

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
row vector = 1 4 5
            
colomn vector= 1
               2﻿
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Finally A can be written as A = row vector * column vector

Similarly for h we have we need to find how to divide this in to simpler form.We can use **"Single Value Decomposition (SVD)"** on h and we can get **U*S*V'=h** (V' is the transpose of V). So by applying SVD on h we get following :

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
U =
   -0.4085    0.9116   -0.0445
   -0.8162   -0.3867   -0.4292
   -0.4085   -0.1390    0.9021

S =
    0.3749         0         0
         0    0.0000         0
         0         0    0.0000

V =
   -0.4085   -0.3497   -0.8431
   -0.8162    0.5534    0.1660
   -0.4085   -0.7559    0.5115﻿

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If you observe decomposed matrices we again see "S" which is rank 1 matrix hence, we can again decompose it into simpler form. We can do **s1*s2=S**. Initially we had U*S*V'=h . So now we have (U*s1) * (s2*V')=h. Thus we are reaching close to our goal of representing h in form of **h = h1 * h2** where, **h1** and **h2** are one-dimensional.

Now to divide **S** we can again chose SVD but as it is very simple matrix so , we can use square root to divide **S** as follows

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
h1 = U*sqrt(S)

h1 =
   -0.2501    0.0000   -0.0000
   -0.4997   -0.0000   -0.0000
   -0.2501   -0.0000    0.0000

h2 = sqrt(S)*V'

h2 =
   -0.2501   -0.4997   -0.2501
   -0.0000    0.0000   -0.0000
   -0.0000    0.0000    0.0000
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

So now we have simplified two-dimensional h into h1 and h1 which are one dimensional.

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
h1 =
    0.2501
    0.4997
    0.2501
h2 =
    0.2501    0.4997    0.2501

>> h1*h2

ans = h
    0.0626    0.1250    0.0626
    0.1250    0.2497    0.1250
    0.0626    0.1250    0.0626
﻿~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

﻿Hence by using **h1** and **h2** we are getting back **h**.

Now we can take one-dimensional filter [0.2501 0.4997 0.2501] and apply it on our image to get effect of original Gaussian blur function we had. Lets say our image had X and Y dimension then we apply this one-dimensional filter on X direction first and then we apply same filter on Y direction to get desired result.
