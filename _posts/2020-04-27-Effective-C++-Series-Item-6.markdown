---
layout: default
slug:  "Effective C++ Series Item 6"
date:   2020-04-27 21:21:00 +0900 //YYY--mm-dd
permalink: /:year-:month-:day/:title
is_post: true
---
Chapter 1, Item 6: Explicitly Disallow the User of Compiler-Generated Functions you do not want.
From the previous chapter, you'll know that C++ auto generates constructors when you don't declare it. 

However, what happens if you want to create unique objects? Read on to find out more. 

<!--more-->

For Example: 

```cpp
class HomeForSale { ... };

int main() {
  HomeForSale h1;
  HomeForSale h2;

  h1 = h2; //attempting to copy h2 into h1

  HomeForSale h3(h1); //attempting to copy h1 to h3
  return 0;
}
```

The above copy code dosen't make sense right? Because 2 houses shouldn't be exactly the same. 
How can we prevent that? In normal circumstances, if we do not want something, we simply do not code it out. However, as mention in [earlier blogpost](https://livinginjapan.github.io/2020-03-28/Effective-C++-Series(ii)), if you don't declare, and somebody tries to call, the compiler will declare it for you. 

How can we prevent other classes from calling a copy constructor / assignment operator?<br>
Answer: `just declare them private.`
1. declaring prevent compiler generating its own function
2. setting it private keep people from calling it. 

It's not entirely foolproof though. A member / friend function can still call a private function: 

```cpp

class A 
{
  private:
    void foo() {}

  public: 
    void cat() {}

  friend void bar();
};

void A::cat(){
  foo(); //calling foo()
}

void bar(){
  A a;
  a.foo(); //calling foo again noooo!!!!!!!
}
```

the trick is to not define the function so that they'll get error at link-time. This is a well established trick where lots of C++ iostreams are using it like this. 

So the final code should look like this:

```cpp
class HomeForSale {
  public:
    ...
  private:
    HomeForSale(const HomeForSale&); //declarations only. 
    HomeForSale& operator=(const HomeForSale&);

};
```

Note: common convention is that you can omit function parameters if you are not using it. 
To optimise it by failing at compile time, declare the copy constructor and copy assignment operator private not in HomeForSale class but base class. 

```cpp
class Uncopyable {
protected:                                   // allow construction
  Uncopyable() {}                            // and destruction of
  ~Uncopyable() {}                           // derived objects...

private:
  Uncopyable(const Uncopyable&);             // ...but prevent copying
  Uncopyable& operator=(const Uncopyable&);
};
```

and now HomeForSale can inherit from Uncopyable

```
class HomeForSale : private Uncopyable {
  ....
};

when compiler try to generate a copy constructor / copy assignment operator, if anybody tries to call it, it will try to call the base class and it will fail because it is private. 

C++ is so.. smart  yet frustrating at the same time. 

Ok thats all for today's learning!

- K.