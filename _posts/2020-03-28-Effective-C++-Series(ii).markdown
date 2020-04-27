---
layout: default
slug:  "Effective C++ Series(ii)"
date:   2020-03-28 22:38:00 +0900 //YYY--mm-dd
permalink: /:year-:month-:day/:title
is_post: true
---
Chapter 1, Item 5: **Know what functions C++ silently write and calls.** <br>
First and foremost, something that confuses the hell out of beginners / me are the C++ constructors. There are 3 different kind of constructors in C++. 
1. default constructor
2. copy constructor
3. copy assignment operator. 


So what are the differences between those 3? 
<!--more-->
Writing a simple C++ Program can tell you so: 

```c++
#include <iostream>
class A {
  public:
    A() { std::cout << "default constructor\n"; }
    A(const A& other) { std::cout << "copy constructor\n"; }
    A& operator= (const A& other) {std::cout <<"assignment\n"; }
};


int main(){
  A a;      //default constructor 
  A b(a);   //copy constructor
  A c = a;  //copy constructor
  A d;      //default constructor
  d = a;    //assignment constructor 
  return 0;
}
```

So, if we were to declare an empty class, i.e. 
class A {};

It is essetiantally the same as follows: 
```c++
class A {
  public:
    A() { ... }
    A(const A& rhs) { ... }
    ~A() { ... }
    A& operator=(const A& rhs) { ... }
}
```
This is what C++ compiler will generate for you. Also, it is only generated when needed (when you invoke them). 
However, as long as you declare a *type* of constructor, compilers won't generate it anymore. For example:

```c++
template<typename T>
class A {
  public:
    A(std::string& name, T val);

  private:
    std::string name_;
    T objectVal_;
};

template<typename T>
A<T>::A(std::string& name, T val)
  :name_(name),
  objectVal_(val)
  {std::cout << "default constructor\n";};

int main() {
  // A<int> a; // this will now fail!	
  std::string name("Mike");
  A<int> a(name, 1); //calls default constructor
  A<int> b(a); //compiler generates this copy constructor 
  return 0;
}
```

we tried copying from object `a to b` this can happen even if we did not specify the copy constructor as compiler will generate one for you.
since we tried copying a.name_ to b.name_, it will use the string copy constructor, and we specified int on objectVal_ so it will initialize by copying bits to b.objectVal_. 


That's all for today chapter in learning what the compiler generates for you. 

See ya! 

- K.