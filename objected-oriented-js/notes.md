Object Oriented Javascript
--------------------------

Scopes and Closures
===================

# Lexical Scope

*Lexical scope* Describes regions in your source code where you can refer to a variable by name without getting access errors.

*Global scope* In simple programs with no functions at all, there is exactly one scope called global scope. Every variable in that problem will be stored there. In some environments, the global scope is shared among different programs.

A new lexical scope is defined everytime you create a new function definition. The brackets define a new lexical scope. Code inside functions can access the function scope as well as broader level scopes, i.e. global scope.

Example 1:

    var hero = aHero();
    var newSaga = function() {
        var foil = aFoil(); 
    };  
    console.log(foil);  // hero, newSaga, foil are accessible inside brackets

Example 2:

    var hero = aHero();
    var newSaga = function() {
        foil = aFoil(); // missing var usually happens by accident
    };                  // foil is in global scope
    console.log(foil);  // technically will work...but we should avoid it

Assigning variables for the first time (i.e. *foil* without keyword *var*) puts the variable into the global scope.

Blocks on **if** statements or looping statements do not create new scope.

## Execution Contexts

When a program runs, it builds a storage system for holding the variables and their values. These in-memory scope structures are called *execution contexts*. Execution scopes are different from lexical scopes, in that they are built as the code runs, not as it's typed. The context for a function will always be created as a child of the context that it was defined in. 

### How variables will be available at runtime within different contexts

A new execution context is created each time a given function runs. The execution context looks a lot like key-value pairs. The similarity between execution context (in-memory scopes) and in-memory objects are deceptively close. They rarely interact with each other by the interpreter and should otherwise be considered as living in different worlds.

To visualize how execution contexts are unique, notice how the output of the function below produces *false*.

    var makeArray = function() {
        return [];
    }
    var array1 = makeArray();
    var array2 = makeArray();
    console.log(array1 === array2); // false - different identities


# Closures

Every function should have access to all the variables from all the scopes that surround it. A *closure* is just any function that somehow remains available after those outer scopes have returned.

### Which of the following techniques could be used to retain access to the saga function objects after the newSaga calls that created them had returned?

    var hero = aHero();
    var newSaga = function() {
        var foil = aFoil(); 
        var saga = function() {
            var deed = aDeed();
            console.log(hero + deed + foil);
        };
        saga();
        saga();
    };  
    newSaga();
    newSaga();

1. passing to setTimeout
2. returning sage from newSaga
3. saving saga to a global var

For example, declare in the global scope **var sagas = [];** to hold all the **saga** functions. Then in the body of the **newSaga** function, push them into the global area. 

For example:
    
    var sagas = [];
    var hero = aHero();
    var newSaga = function() {
        var foil = aFoil(); 
        sagas.push(function() {
            var deed = aDeed();
            console.log(hero + deed + foil);
        });
    };  
    newSaga();
    sagas[0](); // "BoyEyesRat"
    sagas[0](); // "BoyDigsRat"
    newSaga();
    sagas[0](); // "BoyPinsRat"
    sagas[1](); // "BoyGetsET"
    sagas[0](); // "BoyEatsRat"

> Understanding what the interpreter is doing when it interprets your code, and understanding scope and closures helps me in writing concise applications that other develops can read and understand.


this
====

> Every object oriented language has a way to dynamically refer to the current object, parameter **this**.

## What is **this** for?

**this** is an identifier that gets a value bound to it, much like a variable. But instead of identifying the value explicitly in your code, **this** gets bound to the correct object automatically. The interpreter's rules for determining the correct bindings resemble the rules for positional function paramters. There are lots of conceptions out there, so let's clear them up by going through a list of things **this** specifically won't bound to.

### What **this** is not bound to...

    var fn = function(a,b) {
        console.log(this);
    };

When interpreter runs, it hits the function definition it will create a function object in memory {f}.

- *this* is not bound to the function object {f}
- *this* is not bound to an instance of {f} (generally)
- *this* is not bound to an object that happens to have that function as a property
- *this* is not bound to an execution context or scope of the function call
- *this* is not bound to the object created by the literal *this* appears within

### What **this** is bound to...

    obj.fn(3,4);
The function that an object is looked up upon when invoked is what *this* binds to. In other words, the object found to the left of the dot where the containing function is called. This definition of what *this* refers to is useful for 90% of all cases.

When there is no object explicitly invoking a function, i.e.
    
    fn(g,b);
the *global* object is what *this* refers to.

A *second* way of binding *this* comes from the *call* function, i.e.

    fn.call( ,g,b);
We get to override the default binding to the global object and override the left-of-the-dot rule. You can pass in whatever value you want into the first paramter of the argument list, which is what *this* will bind to,

### How will **this** get bound within functions passed as callbacks...

    setTimeOut(fn, 1000);
*this* binds to the global object.

    setTimeout(r.method, 1000);
    var setTimeout = function(cb, ms) {
        waitSomehow(ms);
        cb();   // this binds to the global object, b/c nothing to left of dot
    }


The problem of losing parameter bindings is pretty widespread, since any function like *setTimeout* that takes another function as a callback may actually call that function differently than you expected.

    var fn = function(one, two) {
        console.log(this, one, two);
    };
    var r = {}, g = {}, b = {}, y = {};
    r.method = fn;
    setTimeout(r.method, 1000); // <global>, undefined, undefined
    setTimeout(function() {     //    {}   ,    {}    ,    {}
        r.method(g,b);
        });

Callback functions are inherently designed so that they will be invoked by the system you pass them into. Thus, you generally have little control over what the bindings will be for the parameters of the functions you pass in.

Another way of predicting the output of *this*:

    console.log(this);  // <global>
In new specifications, this is not allowed.

Finally, predicting *this* output with 'new':

    new r.method(g,b);  // {} , {},  {}

Binding of *this* affected by *new* keyword. It binds to a brand new object.


prototype delegation
====================

# Prototype Chains

Ongoing lookup-time delegation (prototype chains) vs one-time property copying.


# Prototype Delegation

    var gold = {a:1};
    log(gold.a); // 1
    log(gold.z); // undefined

    var blue = extend({}, gold);
    blue.b = 2;
    log(blue.a); // 1
    log(blue.b); // 2
    log(blue.z); // undefined

    var rose = Object.create(gold);
    rose.b = 2;
    log(rose.a); // 1
    log(rose.b); // 2
    log(rose.z); // undefined

    gold.z = 3;
    log(blue.z); // undefined
    log(rose.z); // 3


# Object Prototype

The object prototype is a top-level object that every javascript object delegates to - this is where all the basic methods or properties are shared.


# Constructor Property

A help method of the object prototype, which makes it easier to tell what function was used to create a certain object. Like all properties, *.constructor* actually points to a different object that's stored elsewhere.


# Array Prototype

The array prototype also delegates common methods to the object prototype. But the array prototype also has its own versions of properties from the object prototype.


Object Decorator Pattern
========================

    var carlike = function(obj, loc) {
        obj.loc = loc;
        return obj;
    };
    var move = function(car) {
        car.loc++;
    };

When a function takes in an object as an input, and then augments that object with some properties or functionality, that function is a decorator. It is common to use adjectives to name your decorator functions.

The value of *this* is automatically bound to the object to the left of the called method *.move()*. For example:

    var carlike = function(obj, loc) {
        obj.loc = loc;
        obj.move = move;
        return obj;
    }; 
    var move = function() {
        this.loc++;             // (1) this refers to tom at (2)
    };
    var tom = carlike({}, 1);
    tom.move();                 // (2) tom calls .move(), so this binds to tom


Functional Classes
==================

The difference between a decorator and class is that a decorator takes in an object as an input, while a class builds the object. The functions that build the fleet of similar objects that conform to the same interface are called constructor functions. Instances are the objects that get returned from constructor instantiations.

A function is a special object. It is allowed to have properties that are accessed with dot notation.

    var Car = function(loc) {
        var obj = {loc: loc};
        extend(obj, Car.methods);   // not js native
        return obj;
    };
    Car.methods = {
        move : function() {
            this.loc++;
        }
    };

A Javascript class is just a function that is capable of creating other objects.

Prototypal Classes
==================

The default object that comes with every function is stored at the key dot prototype. Prototypal classes are much more performant because you do not have to create new methods in memory. The prototype holds references to the necessary methods, which all objects can delegate to.

    var Car = function(loc) {
        var obj = Object.create(Car.prototype);
        obj.loc = loc;
        return obj;
    };
    Car.prototype.move  = function() {
        this.loc++;
    };

Car.prototype is purely cosmetic. It's simply the same as Car.methods, except that .prototype is built into Javascript. Car.prototype object is a freely provided object for storing things with no special charateristics. Again, .prototype is just a container object.

There is still prototype ambiguity. There's a difference between "Car's prototype is Car.prototype" and "amy's prototype is Car.prototype".

Every prototype object comes in with a .constructor property that points back to the function it came attached to. Car.prototype.constructor is Car itself.

##instanceof

The **instanceof** operator works by checking to see if the right operands .prototype object can be found anywhere in the left operand's prototype chain.

    
    var Dog = function() {
        return {legs: 4, bark: alert};
    };
    var fido = Dog();
    log(fido instanceof Dog); // false

The goal of the prototypal pattern is function sharing via a prototype delegation. If methods were defined inside the constructor, there wouldn't be any reason to delegate the instances to any prototype at all.


Pseudoclassical Pattern
=======================

Pseudoclassical patterns of defining classes is so-called because it attempts to resemble class systems from other languages using a thin layer of syntatic conveniences.

Let's refactor the prototypal class pattern of the Car class to the pseudoclassical pattern. Notice how lines (1) and (3) could be repeated in other prototypal definitions of classes. These are things that pseudoclassical patterns can take care of for us. Using the **new** keyword before the function invocation, that function runs in a special mode called the constructor mode. In that mode, we can expect lines (1) and (3) to run automatically.

Here is the prototypal pattern:

    var Car = function(loc) {
        var obj = Object.create(Car.prototype); // (1)
        obj.loc = loc;                          // (2)    
        return obj;                             // (3)    
    };
    Car.prototype.move  = function() {
        this.loc++;
    };

Below is the how pseudoclassical pattern looks in the **Car** function:

    var Car = function(loc) {
        this = Object.create(Car.prototype); // (a)
        this.loc = loc;                     
        this obj;                            // (b)
    };
    Car.prototype.move  = function() {
        this.loc++;
    };
    var tom = new Car(1);
    tom.move();

With **new**, what actually happens is that your function temporarily runs with extra operations inserted into your function lines (a) and (b) above that you didn't explicitly write. Here's how your refactored code will look:

    var Car = function(loc) {
        this.loc = loc;
    };
    Car.prototype.move  = function() {
        this.loc++;
    };
    var tom = new Car(1);
    tom.move();

In summary, there are 3 class patterns: functional, prototypal, and pseudoclassical. Let's move onto thinking about the slightly more advanced code sharing techniques of superclasses and subclasses. Note that the pseudoclassical pattern is better documented online than functional class patterns.


Superclasses and subclasses
===========================

A class works great if you want toa build a fleet of similar objects. But if you want to create a category of classes that is vaguely similar to another, you utilize super classes. For example, to create a Van and Cop car, they can be specialized versions of a super class Car.   

Here, we proceed with describing how super/sub classes work in the pseudoclassical pattern. We want to run functions in the context we choose, hence the introduction of the **call()** method.

    var Car = function(loc) {
        this.loc = loc;
    };
    Car.prototype.move  = function() {
        this.loc++;
    };
    var Van = function(loc) {
        Car.call(this, loc);
    };
    var tom = new Car(1);
    var bol = new Van(9);

The **call()** method calls a function with a given *this* value and arguments provided individually.

Van.prototype delegates to Object.prototype but not to Car.prototype because we have not set up a connection between Car and Van.

Subclass prototype delegation works like the following:

    var Car = function(loc) {
        this.loc = loc;
    };
    Car.prototype.move = function() {
        this.loc++;
    };
    var Van = function(loc) {
        Car.call(this, loc);
    };
    Van.prototype = Object.create(Car.prototype);   // (1)
    Van.prototype.constructor = Van;                // (2)

Notice that in linking the Van's prototype with Car's prototype, you need to create an object the delegates to Car.prototype. However, doing so causes Van.prototype's constructor to be the Car function. Thus, you need to properly set up Van.prototype's constructor to the Van function. 

Summary
=======

You've gone over scopes, closures, this, and prototype chains. In addition, you learned about function decorators and other code reuse techniques for implementing subclassing over prototype chains.

**Sources** [Summary](https://docs.google.com/document/d/1F9DY2TtWbI29KSEIot1WXRqqao7OCd7OOC2W3oubSmc/pub)