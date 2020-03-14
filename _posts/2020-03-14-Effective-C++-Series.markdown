---
layout: default
slug:  "Effective C++ Series"
date:   2020-03-14 22:38:00 +0900 //YYY--mm-dd
permalink: /:year-:month-:day/:title
is_post: true
---
Chapter 1, Item 4: **Make sure that objects are initialised before used.** <br>
Before I actually started reading this book / writing ANY C++ code, I created a mini project (something to keep track of my finance) to get myself started with C++. 
The closest language I know that is similar to C++ was Java. Naturally, I coded the mini project in a very Java style.  While reading the book / working / finding out more information on C++, I looked back at my project and realised I made the most fundemental mistake in C++.  Initialization and assignments are two **VERY** different things. (duh!)
<!--more-->
 <br><br>
As the title says, in C++, always remember to initialize your code. In C++, initialization isn't guaranteed. If the programs reads a variable that is not initialised, it may stop the entire program or run into undefined behaviour.  There are some rules to describe whether initialization of an object is guranteed or not, it is somewhat complicated, but the bottom line is:
1. If you are in the C part of C++, initialization would incur a runtime cost, and it is **NOT** guranteed to take place. 
2. If you are in non C parts, it is not necessarily guranteed to take place, but initializing a vector from the standard library is guranteed. 

See? It can get so messed up. So the best idea is to **ALWAYS** initialize your objects / variables. 

Here comes the "interesting" part (at least for me). 
So in java, when we initialize an object, what we usually do is this:

```java
public class Account
   {
        // instance variable
       private String name;
 
        //constructor
       public Account(String name) { 
            this.name = name;
        }
   }

Account acc = new("Mike"); // Creating account objects 
```

The above code shows you could create a constructor with `this` as a reference to the current object and now 'acc' object has got an instance variable initialized to "Mike".
So... I was being "smart" and did this to my mini project in C++ too! Following example is a short snippet: 

```c++
Account::Account(boost::gregorian::date date, std::string name){
    this->date = date;
    this->name = name;
}
```

Very...embarrassing. Firstly, `this->` is actually implicitly called in C++ when you try to assign `date` or `momthExpenses`.  Therefore, `this` can be omitted actually. 
Next, and most importantly, did you know: 
```
this->date = date; 
```
IS ACTUALLY AN **ASSIGNMENT** AND NOT AN **INITIALIZATION**? Yup, rookie mistake. 
From the book I quote, <br><br>
`The rules of C++ stipulate that data members of an object are initialized before the body of a constructor is entered.` Hence, date or monthExpenses are actually assigned. No what you intended right? This means that, initialization already took place?
Thats right! Before entering the constructor body, initialization ALREADY took place. Hence, the most recommended & better way to write is as follows:

```c++
Account::Account(boost::gregorian::date date, std::string name) 
    : date(date),  //member initialization list 
      name(name)
    { //this is the constructor body which is now empty.
    }
```

That embarrassing code up there (lets call it Embarrassment 1.0), and this one here (still embarrassing. to be improved Embarrassement 2.0) performs the objective - which is to set a value to date and name. However, Embarrassment 2.0 is better too in terms of performance. The reason is that Embarrassment 1.0 will have to first initialize date and name through a default constructor and then assign new values this means that the **default constructor** work are all going to a waste. While Embarrassment 2.0 gets to avoid that and only initialized that data members which uses **copy constructor**. This is as opposed to **default constructor** and then a **copy assignment operator**.

What about constructors with no parameters? Do we still initialise them? - Short answer is: yes it is recommended. <br>
Long answer is: Compiler will help to call default constructor if you dont initialise them. But if you have a data member with a primitive type (i.e C-style type) like an `int`, and you forgotten to initialise it, you will go have to spend some good time debugging - which can be avoided if you just initailise it. 

```c++
//Initializing empty data members
Account::Account() 
    : date(),
      name()
    { 
    }
```

Das all folks! More Effective C++ Series to come!