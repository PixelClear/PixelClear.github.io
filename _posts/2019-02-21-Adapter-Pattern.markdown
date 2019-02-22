---
layout: post
title:  "Understanding Adapter Pattern!!"
date:   2019-02-21 15:13:36 +0100
categories: technical post
---

![Move Semantics]({{ site.url }}/assets/cpp.jpg){:class="img-responsive"}

**Introduction**
=====================================================================================================================================

Lets say we are desiging application that gives information about birds. 
We might start with public base interface bird like below. 
Thinking of what are common behaviours that we can add in bird we can come up with 
fly, makeSound, swim. After we have finalized the interface we can extend/ inherit it to add specific birds.
To keep the bird pure interface we might add this as pure virtual functions 

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

class Bird 
{
 public: 
 
 void fly() = 0;
}

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Then we go on making public inheritance as we know eagle is bird. So we have 

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

class Eagle : public Bird 
{
public:

 void fly() override
 {
 std:cout <"Eagle Flies high!!";
 }
};

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Now lets think of extending this library because many people wants to see different birds behaviours (pun intended).
We now also have penguines but, if we consider penguines we have problem. The interface of bird is not compatible with penguines as penguines dont fly they swim.
So one solution we might think of is adding **swim()** method to Bird. But that kind of makes our design look bad as some of the classes inheriting Bird will
implement **fly()** and some will implement **swim()**. We loose consistency and we can no longer keep Bird strict interface as collection of pure virtual functions.
Something like following will happen 

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

class Bird 
{
 public: 
 
 void fly() = 0;
 void swim() = 0;
}

class Eagle : public Bird
{
 public: 
  
  void fly() override
  {
   std:cout <"Eagle Flies high!!";
  }
  
  void swim() override
  {
    //Empty implimentation    
  }
  
};

class Penguine : public Bird
{
  void fly() override
  {
   //Empty implimentation    
  }
  
  void swim() override
  {
  std:cout <"Penguine swims!!";
  }
};

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

In cases like above we have type which is not consistent with the interface.
The solution might be to use adapter pattern.
Lets see how we can do that using template and function pointer

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

class Bird
{
  public : 
  virtual ~Bird() {}
  virtual void execute() = 0;
}

class BirdAdapter<class T> : public Bird
{
public : 

  BirdAdapter(T* obj, void (T::*f)())
  {
    birdObject_ = obj;
	birdFunction = f;
  }
  
  ~BirdAdapter()
  {
    delete birdObject_; //Only if you are not using smart pointer 
  }
  
  void execute() override
  {
    birdObject_->(*birdFunction)();
  }
  
private :

T* birdObject_;
void (T::*birdFunction)()  
}

class Eagle
{
 public: 
  
  void fly()
  {
   std:cout <"Eagle Flies high!!";
  }
};

class Penguine
{
public:

  void swim()
  {
  std:cout <"Penguine swims!!";
  }
};

int main()
{
 BirdAdapter<Eagle> eA (new Eagle(), &Eagle::fly);
 eA.execute();
 
 BirdAdapter<Penguine> pA(new Penguine(), &Penguine::swim);
 pA.execute();
 
 return 1;
}

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If you see above we have not consistent interface and also we dont have much hierarchy of inheritance.
We can add as much variety of birds classes as we want without disturbing the the interface.
I hope this convinces us on power of this pattern.