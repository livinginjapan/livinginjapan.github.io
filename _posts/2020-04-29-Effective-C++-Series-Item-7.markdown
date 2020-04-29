---
layout: default
slug:  "Effective C++ Series Item 6"
date:   2020-04-29 15:35:00 +0900 //YYY--mm-dd
permalink: /:year-:month-:day/:title
is_post: true
---
Chapter 1, Item 7: Declare destructors virtual in polymorphic base classes. <br>

I've seen so many virtual keyword in c++ which you don't quite see in other languages. So what is `virtual` ? 
<!--more-->

The chapter begins with going straight into an example of polymorphism. They used TimeKeeper as a base class which has derived classes with different approaches to time keeping. 

```cpp

class TimeKeeper {
  public:
    TimeKeeper();
    ~TimeKeeper(); //destructor if you don't already know

};

class AtomicClock : public TimeKeeper { ... };
class WaterClock : public TimeKeeper { ... };
class WristWatch : public TimeKeeper { ... };
```

Many clients will want to access time without knowing the details on how it's calculated. i.e. `factory function`. 

define factory function: 

```
A function that returns base class pointer to a newly created derived class object 
```

Example:
```cpp
TimeKeeper *ptk = getTimeKeeper(); //dynamically allocated object, on the heap, from TimeKeeper hierachy 

//use ptk here

delete ptk; //release to avoid resource leak 
```

So the idea of factory function is that getTimeKeeper returns a pointer to a derived class object, for example AtomicClock. 

Now, the problem is that getTimeKeeper returns a pointer to a derived class (AtomicClock) that object is being deleted via a base class pointer. a TimeKeeper* pointer, and the base class a non virtual destructor. => DISASTER. 

because C++ specifies that when a derived class object is deleted through a pointer to a base with a non virtual destructor, results are undefined. Usually, the derived part of the object never gets destroyed. i.e. AtomicClock object is not destroyed and destructor would not be run. However, the base class part (TimeKeeper part) would be destroyed, leading to a curious "partially destroyed" object. Excellent way to <b> leak resource </b>, corrupt data structure & spend a lot of time with debuggers. 

To fix the problem: 
give the base class a virtual destructor. then deleteing the derived class object will be what you want. it will delete entire object with its derived class parts. 
 
```cpp
class TimeKeeper {
  public:
    TimeKeeper();
    virtual ~TimeKeeper(); //note virtual keyword here
}

TimeKeeper *ptk = getTimeKeeper();

...

delete ptk; //behaves correctly now
```

Base classes like TimeKeeper generally contain virtual functions other than destructor, because purpose of virtual function to allow customization of derived class implementation. 

If a base class does not contain virtual function, often indicates it is not meant to be used as a bsae class.  If a class is not intended to be base class, making the destructor virtual is usually a bad idea.  

Example:
```cpp
class Point {
  public:
    Point(int xCoord, int yCoord);
    ~Point();

  private:
    int x;
    int y;
}
```

if an int occupies 32 bit, Point object can typically fit into a 64 bit register. Point object can be passed as a 64 bit quantity. If Point destructor is virtual, situation changes. 
Object of that that will increase in size, on a 32-bit architecture, they'll go from 64bit ( for the 2 ints) to 96 bit ( ints + virtual pointer) virtual pointer is to determine which virtual function should invoke on the object. on a 64 bit architecture, they may go to 128 bit because pointers are 64 bits in size on such architecture. Adding a vptr to Point will increase 50-100%. Where Point can no longer fit 64 bit register.

Hence, it is gratuitiously declaring all destructors virtual is just as wrong as never declaring them virtual. 

One good rule of thumb:
`Declare a virtual destructor only if you have at least one virtual function.` 


- K.