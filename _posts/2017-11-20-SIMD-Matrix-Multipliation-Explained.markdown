---
layout: post
title:  "SIMD matrix multiplication explained!!!"
date:   2017-11-20 21:22:36 +0100
categories: technical post
---

![]({{ site.url }}/assets/SIMD.png){:class="img-responsive"}

**Introduction**
=====================================================================================================================================
Any CPU platform has some registers associated with it. Registers are closes memory to CPU which it can directly access.
There are many types or registers depending on type of them like General purpose, Segment ,Control , Instruction pointer and so on.
If the platform is 32 bit registers are also 32 bit sized. Most of the CPU platforms now support registers for doing SSE (Single instruction multiple data Streaming Extensions). Specifically Intel with its P4 came with MMX (multimedial exchange) registers those were of size 128 bit.
Advantage with this registers was that with single load cpu instruction it can store 128 bits of data. If you see it in other way you can store
4 floats in registers in single cpu instruction. If you try to do same with general purpose 32 bit registers it would take 4 cpu load instructions to get it done. So this does saved some cpu cycles.You can write SIMD code in assembly but most implementors of c runtime 
gave high level language function layer on it. 

**Explaination**
====================================================================================================================================
I have been working on graphics projects which needs maths library. I have developed SIMD version of this library.
I am putting here one example function.This function is specifically developed for Intel platforms.
Consider multiplication of 4X4 matrices. You will have 4 elements in each row. In order to get them in CPU registers you will have 4 load instructions. If you are using SIMD instructions you can load 128 bit of data or 4 floats in special registers (MMX) in one load instruction saving CPU cycles.
Similarly you can multiply row1 of Matrix1 and col1 of Matrix2 using one multiply instruction. Overall this saves lots of performance saving CPU cycles. Hope you will enjoy the code.

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

#include<intrin.h>
#include<Windows.h>

typedef struct matrix4X4d
{
  double matrix[4][4];
  
}Matrix4x4D;

void M4x4MultM4x4 (Matrix4x4D *m1, Matrix4x4D *m2,
					  Matrix4x4D *m3)
{
	__m128 otherRow0;
	__m128 otherRow1;
	__m128 otherRow2;
	__m128 otherRow3;
	__m128 newRow0;
	__m128 newRow1;
	__m128 newRow2;
	__m128 newRow3;

    /*
     Working of (_mm_loadu_ps)
﻿﻿
     otherRow0 is 128 bit.So we have four floats in it.
     Following call initializes otherRow0 as follows.﻿
     otherRow0[0] = m2->matrix[0][0] -> 0 to 32 bits
     otherRow0[1] = m2->matrix[0][1] -> 32 to 64 bits
     otherRow0[2] = m2->matrix[0][2] -> 64 to 96 bits
     otherRow0[3] = m2->matrix[0][3] -> 96 to 128 bits
﻿     ﻿*/
    otherRow0 = _mm_loadu_ps(m2->matrix[0]);
    otherRow1 = _mm_loadu_ps(m2->matrix[1]);
    otherRow2 = _mm_loadu_ps(m2->matrix[2]);
    otherRow3 = _mm_loadu_ps(m2->matrix[3]);

    /* 
       ﻿Working of (_mm_set1_ps)
       It will initialize temporary 128 bit second parameter
      ﻿ of function (_mm_mul_ps) to m1->matrix[0][0]
       
     ﻿tempRow[0] = m1->matrix[0][0] -> 0 to 32 bits
     tempRow[1] = m1->matrix[0][0] -> 32 to 64 bits
     tempRow[2] = m1->matrix[0][0] -> 64 to 96 bits
     tempRow[3] = m1->matrix[0][0] -> 96 to 128 bits
﻿    ﻿*/
    
    /*
       Working of (_mm_mul_ps)
       It will multiply two 128 bit rows with 4 floats in 
       ﻿it simultaneously.
      ﻿﻿
       newRow0[0] = otherRow0[0] * tempRow[0] -> 0 to 32 bits
       newRow0[1] = otherRow0[1] * tempRow[1] -> 32 to 64 bits
       newRow0[2] = otherRow0[2] * tempRow[2] -> 64 to 96 bits
       newRow0[3] = otherRow0[3] * tempRow[3] -> 96 to 128 bits

﻿﻿       Similar is the working of (_mm_add_ps)
    ﻿﻿*/
﻿
    /*This is normal 4X$ matrix multiplication logic*/
    newRow0 = _mm_mul_ps(otherRow0, _mm_set1_ps(m1->matrix[0][0]));
    newRow0 = _mm_add_ps(newRow0, _mm_mul_ps(otherRow1, _mm_set1_ps(m1->matrix[0][1])));
    newRow0 = _mm_add_ps(newRow0, _mm_mul_ps(otherRow2, _mm_set1_ps(m1->matrix[0][2])));
    newRow0 = _mm_add_ps(newRow0, _mm_mul_ps(otherRow3, _mm_set1_ps(m1->matrix[0][3])));

    newRow1 = _mm_mul_ps(otherRow0, _mm_set1_ps(m1->matrix[1][0]));
    newRow1 = _mm_add_ps(newRow1, _mm_mul_ps(otherRow1, _mm_set1_ps(m1->matrix[1][1])));
    newRow1 = _mm_add_ps(newRow1, _mm_mul_ps(otherRow2, _mm_set1_ps(m1->matrix[1][2])));
    newRow1 = _mm_add_ps(newRow1, _mm_mul_ps(otherRow3, _mm_set1_ps(m1->matrix[1][3])));

    newRow2 = _mm_mul_ps(otherRow0, _mm_set1_ps(m1->matrix[2][0]));
    newRow2 = _mm_add_ps(newRow2, _mm_mul_ps(otherRow1, _mm_set1_ps(m1->matrix[2][1])));
    newRow2 = _mm_add_ps(newRow2, _mm_mul_ps(otherRow2, _mm_set1_ps(m1->matrix[2][2])));
    newRow2 = _mm_add_ps(newRow2, _mm_mul_ps(otherRow3, _mm_set1_ps(m1->matrix[2][3])));

    newRow3 = _mm_mul_ps(otherRow0, _mm_set1_ps(m1->matrix[3][0]));
    newRow3 = _mm_add_ps(newRow3, _mm_mul_ps(otherRow1, _mm_set1_ps(m1->matrix[3][1])));
    newRow3 = _mm_add_ps(newRow3, _mm_mul_ps(otherRow2, _mm_set1_ps(m1->matrix[3][2])));
    newRow3 = _mm_add_ps(newRow3, _mm_mul_ps(otherRow3, _mm_set1_ps(m1->matrix[3][3])));

    /* Working of (_mm_storeu_ps)
       This will load 4 floats from newRow0 to m3->matrix[0]
       in single machine instruction﻿*/
    _mm_storeu_ps(m3->matrix[0], newRow0);
    _mm_storeu_ps(m3->matrix[1], newRow1);
    _mm_storeu_ps(m3->matrix[2], newRow2);
    _mm_storeu_ps(m3->matrix[3], newRow3);
}

int main()
{
  //Initialize and call multiplication for matrix routine
  return 0;
}

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
