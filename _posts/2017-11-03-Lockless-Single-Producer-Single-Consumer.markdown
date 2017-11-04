---
layout: post
title:  "Multi threaded Single Producer Consumer Lockless!!"
date:   2017-11-03 21:22:36 +0100
categories: technical post
---

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~`
#include <iostream>
#include <Windows.h>
#include <tchar.h>
#include <strsafe.h>
#include <atomic>


/*
   Now my queue structure is as follows 

   node1      ->     node2    ->    node3   ->  node4  -> NULL
   front_ptr        fence_ptr                   rear_ptr

   Now Reader while reading will read fence_ptr->next.
   Writer adds at rear->next.
   All items between fence_ptr are non consumed items.
   All items between fron_ptr and fence_ptr are consumed and will be lazy freed by Writer.

*/
template <typename T>
class LockFreeQueue 
{
private:
  struct Node 
  {
    Node( T val ) : value(val), next(nullptr) { }
    T value;
    Node* next;
  };
  Node* front_ptr;
  std::atomic<Node*> fence_ptr, rear_ptr;
 
public:

  LockFreeQueue() 
  {
    front_ptr = fence_ptr = rear_ptr = new Node( T() );
  }

  ~LockFreeQueue() 
  {
    while( first != nullptr ) 
	{ 
      Node* tmp = front_ptr;
      front_ptr = tmp->next;
      delete tmp;
    }
  }

  void AddToQueue( const T& t ) 
  {
	  rear_ptr->next = new Node(t);
	  rear_ptr  = rear_ptr->next;    // Finally update the atomic variable

	  /*Lazy Cleanup of remaining objet*/
	  while( front_ptr != fence_ptr ) 
	  { 
		  Node* tmp = front_ptr;
		  front_ptr = front_ptr->next;
		  delete tmp;
	  }
  }

  bool RemoveFromQueue( T& result ) 
  {
	  if( fence_ptr != rear_ptr) 
	  {
		  result = fence_ptr->next->value;
		  fence_ptr = fence_ptr->next;    // Finally update fence_ptr
		  return true;
	  }
	  return false;
  }
};

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

I wanted to implement multithreaded solution for single producer- consumer problem.
Initially I came up with mutex version which was pretty straight forward. Then going further there was solution using compare and exchange
like method and make algorithm lock less. Consider the true nature of problem and single producer-consumer to my advantage I tried using
special node as fence between producer and cosumer and turns out I got simpler solution.
int main()
{
	//Create Thread 1 Writer
	//Create Thread 2 reader
	return 0;
}
