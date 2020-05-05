---
layout: default
slug:  "Effective C++ Series Item 9"
date:   2020-05-05 16:30:00 +0900 //YYY--mm-dd
permalink: /:year-:month-:day/:title
is_post: true
---
Chapter 1, Item 9: Never call virtual functions during construction or destruction. <br>
it will likely not do what you expected. 

<!--more-->

```cpp

class Transaction {
  public:
    Transaction();
    virtual void logTransaction() const = 0; //pure virtual ctor
};

Transaction::Transaction() {
  ... //do something
  logTransaction();
}

class BuyTransaction : public Transaction {
  public:
    BuyTransaction();
    virtual void logTransaction() const;
};

class SellTransaction : public Transaction {
  public:
    SellTransaction();
    virtual void logTransaction() const;
}
```

with that sample code above, let's begin with:
`BuyTransaction b;`

What is going to happen?
1. `Transaction` constructor will first be called
2. `BuyTransaction` constructor will then be called


Based class parts are first constructed before derived class parts. I think you might be getting some gist of it now. 

If you were to take a look at `Transaction Constructor`, the last line calls virtual function `logTransaction`, which `logTransaction()` is it going to get called?

Here is the problem: <b>during base class construction, virtual functions never go down into derived classes. </b>  Hence, derived class data memeber have not initialized when base class constructor run. Some co,pilers will issue a warning, some don't.

The same goes for destructor, the derived class destructor will run first and then the base class destructor.  

How can we ensure that proper version of logTransaction is called each time an object in the `Transaction` hierachy is created? 

answer: make logTransaction into non-virtual function in Transaction then require the derived class constructor pass the necessary log information to the Transaction constructor. Example:

```cpp
class Tranaction {
  public:
    explicit Transaction(const std& logInfo);
    void logTransaction(const std::string& logInfo) const; //non virtual

};

Transaction::Transaction(const std::string logInfo){
  logTransaction(logInfo);
}

class BuyTransaction : public Transaction {
  public:
    BuyTransaction(parameters)
    : Transaction(createLogString(parameters))
    { ... }

  private:
    static std::string createLogString(parameters);
}
```

Since you can't use virtual function to call down from base classes during construction, you can compensate by having derived class pass necessary information up to base class constructor. 
- K.