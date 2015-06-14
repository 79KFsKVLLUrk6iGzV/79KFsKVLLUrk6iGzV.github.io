---
title: Reference Counting
---
# Reference Counting

We've already looked at operator overloading and templates, but they can be combined to do some pretty interesting things.

Before we look at how reference counting works, we'll first look at something called garbage collection.

As you might know some languages support garbage collection.  With this feature the programmer is not responsible for deallocating memory, it is done automatically by the computer.  Languages such as Java and C# support automatic garbage collection.

Garbage collection means that as a programmer you don't need to remember to always say delete on every object that you new.  **TODO: explain why this is important**

## Garbage Collection

How does garbage collection work?

Here's the basic idea:

We can imagine all the objects in our program as nodes.  Each pointer represents an edge.  If we know about all the nodes and all the edges (not always an easy problem) then we can think of memory as a graph.

We'll start with a set of root objects.  These are the objects that are currently accessible by the program.  These would include things like local variables in the current function as well as global variables.

Periodically program execution will pause.  At this point a runtime system will traverse the entire graph, starting with the root nodes.  Each time a node is encountered it is marked as in-use.  This is known as the *trace*.

![](gc-1.png)

At the end of the trace any nodes that have not been marked as in-use are deleted or *sweeped*.  Garbage collectors are usually known as *trace and sweep* collectors.

Note that everything that the trace and sweep process cleans up is definitely garbage.  If the object cannot possibly be reached via any of the root nodes then it must be garbage.  The opposite is not always true.  Sometimes there is garbage that can be reached.

One way that this could happen is if we had a list.  If we keep adding elements to the list and we keep a reference to the list itself, then none of the list nodes can be considered garbage by the trace and sweep algorithm.  But if we never actually use those list elements then they are as good as garbage.

Even if we did use them, a program will not be able to add elements to a list forever.  Eventually the computer will run out of memory.  The key take away here is that garbage collection is not a magic bullet.  Memory leaks can still be a problem in garbage collected languages--they just will tend to be a problem much less frequently.

### Building the Graph

Conceptually trace and sweep is all well and good, but in reality it doesn't actually work very well for C/C++ programs.  The core issue is that it is actually somewhat difficult to know where all the nodes are, and even more difficult to know what all the edges are.

We know that edges would be pointers, but what is a pointer really?  It is just a number.  There isn't a very good way for a garbage collector to look at a generic C++ object in memory and reliably identify the pointers.  There are libraries that [attempt](http://www.hboehm.info/gc/) to do this.

These garbage collectors for C++ are known as conservative garbage collectors.  This isn't due to their political views, it is because they assume that an address that looks like a pointer probably is.  This means that some objects that should be cleaned up will not be.

Languages like Java or C# that are built around the idea of a garbage collector can allow give the garbage collector a more precise understanding of the object graph.

### Performance

Historically the performance of garbage collectors was not very good.  In the 80s and 90s many somewhat popular garbage collected languages suffered from performance problems.  A large part of the performance issue were related to the garbage collector needing to pause the program to do the object graph trace.  These performance issues hindered the popularity of garbage collection.

As computers have gotten faster, memory has grown larger, and garbage collectors have gotten better many of the performance disadvantages of garbage collectors have been reduced or eliminated.

One big performance improvement is in the collectors themselves.  They typically use something known a 'generational' garbage collection.  The basic idea is that object that have been around for a long time are likely to stay around for a long time and object that were recently created are likely to die soon.  As an object survives more trace and sweep cycles the object is promoted to a higher level.  Objects at a higher level then become sweeped less frequently.  This makes tracing the object graph much quicker because the side of the graph is smaller.

Another big reason for garbage collectors working better is improvements in available memory.  If a computer has plenty of extra memory, then it is easier for a garbage collector to to collection less frequently.  Programs will typically use more memory than they actually need, but as long as there is plenty of memory this isn't a problem.  Typically if there is at least 5 more memory available than the program actually needs then garbage collection will have a minimal impact on performance.  With 3 times more memory it is 17% slower.  With twice as much it is 70% slower.