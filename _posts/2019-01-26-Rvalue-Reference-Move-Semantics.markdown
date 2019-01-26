---
layout: post
title:  "Rvalue reference and Move semantics!!"
date:   2019-01-26 15:13:36 +0100
categories: technical post
---

![Move Semantics]({{ site.url }}/assets/cpp.jpg){:class="img-responsive"}

**Introduction**
=====================================================================================================================================
Before actually looking into "what is rvalue and rvalue references" are we should look into what problem they are trying to solve.

**Implementing Move Semantics**
=====================================================================================================================================
Suppose there is no rvalue references.
Lets see how assignment operator will look like and how it will behave

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
X& X::operator=(X const& rhs)
{
  // [...]
  // Make a clone of what rhsrefers to.
  // Destruct the resource that . 
  // Attach the clone to this.
  // [...]
}
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

**Note:** here that it is accepting & of x.

Now lets see what happens to following code with above declared operator=

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
X foo();
X obj;

obj = foo(); ---> (1)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

(1) above is converted to call like obj.=(foo()).
As return value of foo() is passed as parameter to operator= and we cant take address of return value of function.
So its rvalue( dont forget we dont dont yet have rvalue references).

So following thing will happen

a. tmp object will be created and return value of foo will be copied into it and = will be called with ref to tmp.
b. Delete resources held by this (obj)
c. copy resources from tmp to obj
d. destruct tmp obj

But we have return value of foo() and obj is in hand.
So, can we somehow avoid this tmp creation and init x directly with return value of foo()?

**Note:** 
we were not able to call operator= as operator= as it takes reference as input.
But, foo() is rvalue and we dont have rvalue references yet.

Here comes to our rescue 'rvalue references'.

So if we have X as type then X& is lvalue reference and X&& is rvalue reference.
As we need to distinguish the move assignment from normal assignment we will use X&& to overload assignment.

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
X& X::operator=(X&& rhs)
{
 //Exchange the content between *this and rhs
}
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

**Is rvalue reference lvalue or rvalue?**
=====================================================================================================================================
Rvalue reference can be rvalue or lvalue.
If rvalue reference has name then it is lvalue.
Consider following code 

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
void foo(X&& x)
{
  X anotherX = x; // calls copy constructor version X(X const & rhs)
}

otherwise rvalue reference is rvalue consider following code 
X&& foo();
X x = foo(); // calls move constructor version X(X&& rhs) because the thing on
            // the right hand side has no name
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

**Forcing move semantics**
=====================================================================================================================================
As per rule above remember we can force move semantics when needed 
consider following code

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
class Base
{
//......................
Base(Base const & rhs); // non-move semantics
Base(Base&& rhs); // move semantics
//.......................
}
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If we do like following in derived class its not wrong per say but will invoke copy constructor

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
class Derived : public Base
{
Derived(Derived const & rhs) 
  : Base(rhs) wrong: rhs is an lvalue remember point above.
  {
   // Derived-specific stuff
  }
}

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

So right way is 

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
class Derived : public Base
{
  Derived(Derived&& rhs) 
  : Base(std::move(rhs)) // good, calls Base(Base&& rhs)
  {
   // Derived-specific stuff
  }
}
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

by using std::move() function provided in std lib we are forcing lvalue to be rvalue reference. 
Hence initiating proper move semantics constructor of Base.

**Note:** 
We have to be carefull particularly forcing std::move() on return values of function.
Consider following code

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
T f()
{
  T obj;
  
  return std::move(obj);
}

T t = f();
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Here thinking that we are returning by value so, it will be better to move value as optimization could lead to slower code.
By forcing move semantics you are not letting compiler do NRVO. 
Compiler when using NRVO will never call move constructor hence it will be faster than what we have done.
We will see details about RVO/NRVO in next series of articles.

**When move constructor will be called?**
=====================================================================================================================================
Move constructor will be called in following cases 

a. initialization: T a = std::move(b); or T a(std::move(b));, where b is of type T;
b. function argument passing: f(std::move(a));, where a is of type T and f is void f(T t);
c. function return: return a; inside a function such as T f(), where a is of type T which has a move constructor.

**Note:** 
Point (c) will happend only if there is no RVO or NRVO done by compiler.
We will see details about RVO/NRVO in next series of articles.

**Move constructor and throwing Exceptions?**
=====================================================================================================================================
It is point to remember that when you define your own move constructor for class it should throw exceptions.
To make strong exception guarantee possible, user-defined move constructors should not throw exceptions. 
For example, std::vector relies on std::move_if_noexcept to choose between move and copy when the elements need to be relocated.
So, if your move constructor throws exception consider following code 

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
std::vector<T> f()
{
  std::vector<T> vec= { .... };
  
  return vec;
}
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

**Note:** 
you might expect move constructor for T will be used here (Assuming no NRVO).
but if your move constructor throws exception copy constructor will be used internally.