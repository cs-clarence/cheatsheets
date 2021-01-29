# <center> C++ FAQs </center>

## QUESTION: Are static members inherited?

* **ANSWER**: In terms of visibility, yes (as long as it's visibility is set to `public` or `private`). 

But a `static` member variable itself will not be duplicated in a derived class in case where multiple inheritance of base class is involved.

* **TIP**: If a classes in a hierarchy needs it's own version (inherit a copy) of a `static` member variable in each level (i.e. `instances_` in the example below) set it's visibility to private as derived classes don't need to mutate that variable. You can then make static method to access that static variable.

It's possible to still set the static member variable to public or protected 
and still access it (via fully qualifying it's name i.e. `Base::instances_` ) but that doesn't protect the variable from being accidentally modified by clients. 

``` 

// header file
class Base
{
public:
    // increment instances_ when constructed
    Base() { ++instances_; }   

    // decrement instances_ when destructed
    // declare destructor pure virtual, defined in implementation file
    virtual ~Base() = 0; 

    // so the outside world (and derived classes)
    // can access it's instances_ variable
    static uint64_t instances() { return Base::instances_; }

private:
    // derived classes may access (depending on it's visibility) this 
    // but they don't get a copy of this variable
    static uint64_t instances_;    
};

// implementation file
uint64_t Base::instances_{};

// implementation for pure virtual destructor is needed
// even if it does nothing
// because a derived class will call it's base's destructor no matter what
Base::~Base()
{ --instances; }

// .................................

// header file
class Derived : public Base
{
public:
    // increment it's instances_ when constructed
    Derived() { ++instances_; }   

    // decrement instances_ when destructed
    ~Derived() { --instances_; } 

    // so the outside world (and derived classes)
    // can access it's instances_ variable
    static uint64_t instances() { return Derived::instances_; }

private:
    // also tracks it's instances
    static uint64_t instances_;    
};

// implementation file
uint64_t Derived::instances_{};

// other various implementations
```

---

## QUESTION: When are full class definitions needed? When will forward declaration suffice?

* **ANSWER**: Anywhere where a `class`' size is needed to be known and when a statement requires an object constructor, full class definition is needed (`#include` the header file). For everything else, class declaration is enough. Consider the following:

``` 

// EXAMPLE 1
class Bar;  // just a forward declaration, no inclusion of header file

class Foo   // Foo is incomplete until the closing bracket is reached
{
public:
    // public interfaces
    
    // return types
    Bar getBar()    // OK: Declaration doesn't need to know it's
                    // return type's size

    Bar getBar()    // ERROR: in the implementation 
    {}              // return type needs the class full definition
                    // because the function's implementation needs to
                    // know it's return type's size

    // parameters
    void setBar(Bar b); // OK: Parameters in function declaration
                        // don't need to know the type's size

    void setBar(Bar b)  // ERROR: parameters in function implementation
    {}                  // needs to know the type's size
                        // and it's constructors
    
    // SUMMARY: forward class declaration is sufficient 
    // for function prototypes, but not for implementations
    // HOWEVER: if a function with an inline definiton within the class
    // calls a a function (with just a declaration at the time of the call)
    // that has a return type or parameter of an imcomplete class 
    // (just forward declaration), the compilation will fail
    // A full class definition is also needed in that case

    void callGetBar()
    {
        this->getBar()  // ERROR: getBar() returns an incomplete type
                        // include full class definition (header file)
                        // OR implement this function outside the class
                        // (i.e. in a .cpp file)
    }

    // EXCEPTIONS TO THE RULE:
    // Methods with return types and parameters types of the enclosing
    // class are OK even with their inline implementation
    Foo getFoo()        // OK: methods getFoo() returns an object of
    {}                  // enclosing class
    void setFoo(Foo)    // OK: method setFoo() takes in an argument object 
    {}                  // of the enclosing class

private:
    // data members

    // other type
    Bar bar;    // ERROR: needs a constructor and the type's size
                // so include the full class definition

    Bar &barRef = bar;  
                // OK: (assuming that the statement above is legal)
                // References also don't need to know the size of the 
                // object they will refer to 
                // ('cause they are just pointers)

    Bar *pBar = &bar    
                // OK: pointers are ok

    // same type
    Foo foo;    // ERROR: foo is incomplete, not legal
                // Foo will only be complete once 
                // outside the class bracket

    Foo *pfoo   // OK: Pointers don't need
                // to know their type's size
                // they also don't need a constructor

    Foo &rFoo;  // OK: References are ok

    Foo *pfoo2 = new Foo{}; 
                // ERROR: operator new needs to know
                // the type's size to allocate memory in the heap
                // Foo{} also invokes a default constructor
}

```

---

## QUESTION: Can you overload a function from a base class in a derived class?

* **ANSWER**: No. If you re-declare the same name from a base class, the name will hide the one in the base class. It doesn't matter if it's a different signature, or if it's a variable, an identifier in a current scope will the one in the outer scope. 
* In fact, hiding is not limited to functions; variables, nested classes, enums, structs, unions, macros, type aliases and whatever will re-declare a name/identifier in a scope, will hide the one in the outer scope, even if they're not the same kind of symbol (a symbol can be a name for a variable, class, enum, or anything kind of identifier). Consider the following:

``` 

struct Base
{
    int add(int x, int y)
    {
        std::cout << "adding int" << std::endl;
        return x + y;
    }
    char add(char x, char y)
    {
        std::cout << "adding char" << std::endl;
        return x + y;
    }
};
struct Derived final : public Base
{
    // this, on the surface, works, but not as you expected
    // this will not overload, it will actually hide the versions
    // of add() in the base class
    // you can only overload in the same scope,
    // but Derived does not have the same scope as Base
    // Base is actually an outer scope of Derived's scope
    double add(double x, double y)  
    {
        std::cout << "adding double" << std::endl;
        return x + y;
    }

    // QUESTION: How to allow overloading of a function in the Base?
    // ANSWER: bring the Base::add() to Derived's scope
    // adding the following code will fix the problem
    // using Base::add;
    // this works, but if you do have to use this,
    // you probably have a problem in your design
};

int main()
{
    int x{1}, y{2};
    char c1{1}, c2{2};
    double d1{.1f}, d2{.3f};

    std::cout << Derived{}.add(x, y) << std::endl;      
                // actually calls add(double, double)
                // implicitly converts int to double

    std::cout << Derived{}.add(c1, c2) << std::endl;
                // actually calls add(double, double)
                // implicitly converts char to double

    std::cout << Derived{}.add(d1, d2) << std::endl;
                // actually calls add(double, double)

    // just as names in the current scope will hide the names in 
    // the outer scope,
    // names in the Derived class scope will hide the names 
    // in the Base class scope
}
```

---

## QUESTION: Does bringing a name/identifier from another namespace/scope the same as declaration?

* **ANSWER**: No. Declaration means introducing a name that has a definition existing somewhere. Using the `using` (i.e. `using Base::add;`) keyword to bring a name into a local scope does not declare a new name but will only allow to refer to name without full qualification. It allows the clients to refer to `Base::add()` as just `add()` by bringing that name into the local scope by typing <nobr>`using Base::add;`</nobr> in the current scope. 

<br />

* Another difference is using the `using` keyword to bring a name into a scope does not ***hide*** names from outer scopes but can cause conflict.

<br />

* Declaration doesn't cause conflict because it hides names in the outer scope if it has the same name. 

---

## QUESTION: Does private inheritance of a class restrict the internal use in the derived one?

* **ANSWER**: No. Private inheritance does not make the members of a base class inaccessible in a derived class, it just makes all the members of a base private in the derived. It means derived can still access all public and protected members but the outside cannot access those functions through the derived class. Consider the following:

``` 

class Base
{
public:
    virtual void memFun1() = 0; // pure virtual public method
    virtual void memFun2() = 0; // pure virtual public method
    virtual void memFun3() = 0; // pure virtual public method
};

void Base::memFun1()    // definition of pure virtual method in base
{
    std::cout << __func__ << std::endl;
}

class Derived : private Base    // all members of Base are now private here
{
    void memFun1() override     // forward the call to Base's version
    {
        Base::memFun1();        // still have access to memFun1() in Base
    }
};

int main()
{
    Derived{}.memFun1();        
                    // OK: made a forwarder function to memFun1() in Base

    Derived{}.memFun2();        // ERROR: memFun2() is privately inherited
    Derived{}.memFun3();        // ERROR: memFun3() is privately inherited
}
```

#### <br/>

## QUESTION: Can you call an implemented pure virtual function from a derived class?

* **ANSWER**: Yes, you just need to fully qualify the call.

``` 

class Base
{
    virtual uint64_t getObjectId() = 0; // pure virtual
}

uint64_t Base::getObjectId()            // implementation for pure virtual
{ /* some code */}

class Derived : public Base
{
    uint64_t getObjectId() override
    { /* implementation for Derived's version */ }
}

int main()
{
    Derived{}.Base::getObjectId();      
                // fully qualified call to Base's version of a
                // pure virtual method

    Derived{}.getObjectId();    // call Derived's version of getObjectId()
}
```

---

## QUESTION: Can you redefine a default value/argument of a parameter?

* **ANSWER**: Yes you can. But the result might not be what you expected.

Consider the following:

``` 

struct Base
{
    virtual int getInt(int = 0) = 0;
    // NOTE: virtual functions are statically binded
    // but their default parameter values are not
};

struct Derived : public Base
{
    int getInt(int i = 1)
    { return i; }
};

auto main() -> int
{
    Base *pBase = new Derived{};
    std::cout << pBase->getInt() << std::endl;  // OUTPUT: 0

    Derived *pDerived = new Derived{};
    std::cout << pBase->getInt() << std::endl;  // OUTPUT: 1

    // The reason for this behavior is the default argument to be chosen
    // is the one in the implementation of their static type
    // The static type of pBase is: Base*
    // while it's dynamic type is Derived*
    // The default argument will come from Base

    // The static type of pDerived is: Derived*
    // and it's dynamic type is also Derived*
    // The default argument will come from Derived

}
```

---

## QUESTION: What does private and protected inheritance imply?

* **ANSWER**: In C++,  `private` and `protected` inheritance implies "is-implemented-terms-of" relationship as opposed to `public` inheritance which implies "`is-a`" relationship.
* The "`implemented-in-terms-of`" and "`has-a`" relationship are not supposed to be seen as *inheritance* but as *composition*. A class that privately or protectedly inherits from a base class reuses the base's members' implementations internally, but that's it, the user of the derived class doesn't need to know that derived inherits from base.
* Private and protected inheritance prohibits implicitly casting a derived to base, neither by value, by pointer and by references. This happens because private and protected inheritance isn't supposed to be polymorphic. Explicitly casting will work through pointers and references by C-Style casts and `reinterpret_cast<>()`, but not by value.

``` 

struct Base
{
    virtual int getInt(int = 0) = 0;
};

struct Derived : private Base
{
    int getInt(int i = 1)
    {
        std::cout << "Actually from Derived" << std::endl;
        return i;
    }
};

struct SomeOtherClass : private Derived
{

};

auto main() -> int
{
    Base *pBase = new Derived{};            // ERROR: cannot implicit casts
                                            // pointer-to-Derived to 
                                            // pointer-to-Base

    Base *pBase2 = (Base *) new Derived{};  // OK: Explicitly casted 
                                            // through C-Style cast
    
    Base *pBase3 = reinterpret_cast<Base *>(new Derived{});
                                            // OK: Explicit cast through 
                                            // reinterpret cast

    Base *pBase3 = static_cast<Base *>(new Derived{});
                                            // ERROR: Explicit pointer 
                                            // cast through static_cast 
                                            // cast

    Base *pBase3 = dynamic_cast<Base *>(new Derived{});
                                            // ERROR: Explicit pointer 
                                            // cast through dynamic_cast 
                                            // cast

    Derived d;
    Base &rBase = (Base &) d;               // OK: Explicitly reference 
                                            // casting through C-Style cast

    SomeOtherClass s;
    Derived vDerived = s;                   // ERROR: cannot implicitly 
                                            // cast an object of 
                                            // SomeOtherClass to Derived

    Derived vDerived = (Derived) s;         // ERROR: cannot explicitly 
                                            // cast an object of 
                                            // SomeOtherClass to Derived
}
```

* With that said, inheritance is still inheritance, even if it's not supposed to be an "`is-a`" relationship. Even if the compiler prohibits implicit casting, it doesn't prohibit polymorphic calls to a virtual function through a base class pointer. Consider the following:

``` 

struct Base
{ virtual int getInt(int = 0) = 0; };

struct Derived : private Base
{
    int getInt(int i = 1)
    {
        std::cout << "Actually from Derived" << std::endl;
        return i;
    }
};

auto main() -> int
{
    Base *pBase = (Base *) new Derived{};   // OK: explicit pointer cast
    pBase->getInt();        // OUTPUT: Actually from Derived
}

```

---

## QUESTION: In what case should you use virtual inheritance? 

* **ANSWER**: Use virtual inheritance judiciously. Other than a class hierarchy with multiple inheritance, there is no other use for virtual inheritance.
* Use virtual inheritance on a base class in which it's members should not be duplicated.

---

## QUESTION: Is there an overhead to virtual inheritance?

* **ANSWER**: Yes. A derived class that is directly or indirectly inherting a base class virtually will need to keep track of what which of it's bases are virtual. This derived class will be bigger than if it wasn't virtual inheriting. Acces to virtual base will be slower. Assignment to a class that uses virtual inheritance will be slower. Every class in a hierarchy that uses virtual inheritance will also be affected, no matter how distant they are from the virtual base. The actual amount of overhead is compiler implementation dependent.

---

## QUESTION: What happens when operator `new` can't allocate enough memory?

* **ANSWER**: When operator `new` can't fulfill a memory allocation request, it will throw an exception of an object of type `std::bad_alloc`.
* That behaviour can be changed by creating a *new handler function* that has the signature `void()` and passing it to `std::set_new_handler()` (it is in `<new>` header). `std::set_new_handler()` takes in a pointer-to-function returning void (`void (*func)()`), and returns also a pointer-to-function returning void (`void (*func)()`).
* There is no default `new handler function`, so when operator new can't allocate enough memory, it will check if the current pointer to a `new handler function` is pointed to a `nullptr`, and if it does, it will not call it and instead throw an exception object of type `std::bad_alloc`.
* `void (*)()` is `typedef`ed as `new_handler` in the `namespace` `std`.
* When `std::set_new_handler()` is called, it returns the function that was in effect (the previous handler, default is `nullptr`) before it was called.
* **NOTE**: The new_handler will be called repeatedly until the resources required are fulfilled. The default mechanism of `new` will throw an exception to terminate the program, so if you want customize it so it doesn't terminate, you should set a `new_handler` that should be able to free memory so `operator new` can allocate it or if it's impossible to allocate, have it call `std::terminate()`. 

---

## QUESTION: What's the difference between `std::terminate()` , `std::exit()` , `std::quick_exit()` and `std::abort()` ?

* **ANSWER**: Just as you can break out of a loop using `break`, you can break out of a program using `std::terminate()`,  `std::exit()` and `std::abort()`. Unlike `return`, which just jump out from a function, the three cause the program to terminate immediately. The description for each them is the following: <br/><br/>
* **`std::exit()`** - A function that terminates a program in the runtime-level (the program does partial clean-up), takes in an integer error code that the operating system can use to determine if the program exited successfully. Just like `return 0` in the function `main()`,  `std::exit(0)` also means the program ended successfully. Calling `std::exit(0)` in `main()` in place of `return 0` has the same effect. 

<br/><br/>
The good thing about `std::exit()` is that it can be called in any function so the control doesn't have to be in `main()` before the program can exit if something is wrong in the program. 
<br/><br/>
`std::exit()` can also call a correspoding *exit handler function* (or multiple *exit handler functions*) before the program completely terminates by using the `std::atexit()` function that takes a pointer-to-function with the signature `void()` , returns `0` when it successfully sets the function, returns non-zero otherwise. If multiple functions are set using `std::atexit()` , the order of execution is the first function that was set is last and the last function that was set is first. (last in, first out (**LIFO**) order of execution).
<br/><br/>
Consider the following:

``` 

void doCleanUp1()   // will get called on pre-termination of program
{ /* do program cleanup */ }
void doCleanUp2()   // will get called on pre-termination of program
{ /* do program cleanup */ }

int global{};           // destructor will get called

void criticalFunction()
{
    // do critical stuff
    if (/* system critical stuff failed */) 
        std::exit(1);       // end the program with error code 1
}

int main()
{
    std::atexit(&doCleanUp1);
                        // set the function to call when
                        // the program invokes std::exit()
                        // or when main() returns
                        // 

     std::atexit(&doCleanUp2);
                        // will get called before doCleanUp1
                        // (LIFO order)

    int x{};            // destructor won't get called
    criticalFunction(); // control doesn't need to come back to main()
                        // criticalFunction() failed

    return 0;           // program will only return 0 if criticalFunction()
                        // succeeded
}
```

<br/><br/>

* **`std::quick_exit()`** - Exactly like `std::exit()`, takes an integer as error code and implies normal program termination but, unlike `std::exit()`, without **ANY** clean-up, not even `static` objects. 

<br/> <br/>
**`std::quick_exit()`** can also set a corresponding *quick exit handler function* by using `std::at_quick_exit()` . It has the same signature as `std::atexit()` (takes pointer-to-function with `void()` signature, returns zero when successful and non-zero otherwise), can set multiple quick exit handlers (order of execution is **LIFO**), but the difference is that quick exit handlers only get called when `std::quick_exit()` is called. When `main()` returns, it only calls functions set by `std::atexit()` , and not by `std::quick_exit()` . 
<br/><br/>
Consider the following:

``` 

int global{};           // destructor will not get called

void criticalFunction()
{
    // do critical stuff
    if (/* system critical stuff failed */) 
        std::quick_exit(1);       // end the program with error code 1
}

void doCleanUp1()   // will get called when std::quick_exit() is called
{ /* do program cleanup */ }

void doCleanUp2()   // will get called when std::quick_exit() is called
{ /* do program cleanup */ }

int main()
{
    std::at_quick_exit(&doCleanUp1);
                        // set the function to call when
                        // the program invokes std::at_quick_exit()

     std::at_quick_exit(&doCleanUp2);
                        // will get called before doCleanUp1
                        // (LIFO order)

    int x{};            // destructor won't get called

    criticalFunction(); // control doesn't need to come back to main()
                        // when criticalFunction() failed

    return 0;           // program will only return 0 if criticalFunction()
                        // succeeded
}
```

<br/><br/>

* **`std::abort()`** - implies abnormal program termination, it raises SIGABRT signal on POSIX systems when called, immediately terminating the program. Like `std::quick_exit()`, program will not get an opportunity to do any cleanup, no handler functions get called, no object destructors will get called.

<br/><br/>
Consider the following:

``` 

int gX{};           // will not get destructed

int main()
{
    int lX{};       // will not get destructed

    std::abort();   // terminates the program prematurely

}
```

<br/><br/>

* **`std::terminate()`** - automatically gets called when an unhandled exception occurs. Can be called manually. When called, it calls a *terminate handler function* (the default is `std::abort()`). A custom *terminate handler function* can be set by calling `std::set_terminate()`, it that takes a pointer-to-function with a signature `void()` and returns the a pointer to the previous terminate handler function (same signature as the parameter). `std::terminate()` will only call one *terminate handler function* unlike `std::exit()` and `std::quick_exit()`, both of which can call multiple exit handlers and quick exit handlers respectively.

<br/><br/>
Consider the following:

``` 

static void (*ptrToPrevious)();
void terminateHandler() // will get called when std::terminate() is called
{
    std::cerr << "Premature program exit" << std::endl;
    ptrToPrevious();
}

int main()
{
    ptrToPrevious = std::set_terminate(&terminateHandler);
                        // replace current terminate handler 
                        // with terminateHandler()
                        // assign previous terminate handler
                        // to ptrToPrevious

    std::terminate();
}
```

<br/><br/>
**DESTRUCTION OF OBJECTS**: 
<br/>
<br/>

* **`std::exit()`**: Only objects with `static` storage duration (such as global objects, static data members, and local static objects) will be destroyed, stack-allocated objects (such as local non-static objects) are not destroyed, operating system handles the rest of the cleanup.

<br/>

* **`std::quick_exit()`**: No destructors are called, operating system handles all the clean-up.

<br/>

* **`std::abort()`**: No destructors are called, operating system handles all the clean-up.

<br/>

* **`std::terminate()`**: No destructors are called, operating system handles all the clean-up.
* **`normal return from main()`**: `static` and stack-allocated objects are destroyed.

<br/><br/>
**SUMMARY**: <br/>

* **`std::exit()`** exits a program as if it `return`ed from `main()`, which implies normal program termination, but only the static objects are destroyed. <br/>
* **`std::quick_exit()`** is exactly like `std::exit()` but without clean-up. <br/>
* **`std::abort()`** implies abnormal program termination, which can be caused by unhandled exceptions, in which the operating system handles the program termination.<br/>
* **`std::terminate()`** - get's called when an unhandled exception occured, calls a function <nobr>(default is `std::abort()`)</nobr> to handle program termination.

---

## QUESTION: How does `free()` in C and `operator delete[]` know how much memory to free?

* **ANSWER**: When you allocate memory using `operator new[]`, a header object (an integer that knows the size of the array) gets added before the actual data of the array. This means the actual size of memory is a bit bigger than the array (additional 4 or 8 bytes, depending on the implementation). When `operator delete[]` gets called, it consults that header object to determine how much memory to free. The same thing happens you `malloc()` or `calloc()`, it has a header object to store how many bytes an object occupies and `free()` consults that header.

---

## QUESTION: 

* **ANSWER**:

---

## QUESTION: 

* **ANSWER**:

---

## QUESTION: 

* **ANSWER**:
