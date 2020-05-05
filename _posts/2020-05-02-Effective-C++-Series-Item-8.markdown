---
layout: default
slug:  "Effective C++ Series Item 8"
date:   2020-05-02 20:35:00 +0900 //YYY--mm-dd
permalink: /:year-:month-:day/:title
is_post: true
---
Chapter 1, Item 8: Prevent exceptions from leaving destructors. <br>

C++ dosen't prohibit destructors from emitting exceptions, but it discourages the use of it - why? 
<!--more-->

```cpp

class Widget {
  public:
    ~Widget() {.... }; //assume it throws an exception

};

void doSomething() {
  std::vector<Widget> v;
  ... 
} //v is automatically destroyed here
```

when V is destroyed, it is responsible for destroying all Widget objects in v, but say the first object went in the destructor and it threw an exception, the other 9 has to be destroyed too otherwise resource will leak. assuming second widget throws an exception again, so now there are two exceptions thrown which is too many for C++ - program will terminate or yield undefined behaviour. Any standard library (list, set) or containers / arrays will result in same behaviour. So try to avoid it. 

But what should you do if you need destructor to perform an operation that may fail by throwing exception?

you can pass the ownership to the caller by providing a regular function (non virtual) for it. 

- K.