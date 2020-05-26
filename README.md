# JS OOPS: The Definitive Guide

This guide will help you understand how OOPS work specifically while building your site/app using JavaScript.

### Object Fundamentals

JavaScript has two categories of data types:
- Primitive
	- undefined
	- null
	- boolean
	- string
	- number
	- object
- Special Objects
	- Arrays
	- Functions
	- RegExp

Objects are like dictionaries. They have a key value pair. The key is always a string. We can assign any datatype to this property.

**Primitives** are passed by value.

    // These two values are not connected. Any changes to num2 will not change num1
    const num1 = 1;
    const num2 = num1;

**Objects** are passed by reference

    // obj1 and obj2 are connected
    const obj1 = { a: 1 };
    const obj2 = obj1;
    obj2.a = 2;
    
    console.log(obj1);
    // object.b will return undefined. You can assign undefined too
    console.log(obj2.b);
    
    // Note: assigning undefined doesn't the property. We use delete keyword for deleting the property
    delete obj2.a;

### Functions & Methods

Functions are object in JavaScript. When you create a function, JavaScript creates an object with 3 properties:
- Function name (e.g "func1")
- The length of the parameters (e.g 2)
- Prototype (we'll discuss this later)

You can do everything with them you do with the normal object. 

    function func1() {
	    return "Hello, World!";
    }
    
    // You can assign properties to them
    func1.a = 1;
    
    // You can assign them to a variable
    var func2 = func1;

`func2` is referencing to the same function. As we discussed earlier, we passed a reference. Hence, now you can call the function `func2` and expect the same results of `func1`

When you put a function inside of a object, it's called a method:

    var obj = {
	  get: function() { return this.val; },
	  val: "Hello, World!"
	};
    
    // Call it like so and when you call like so, the this key word is set to the object
    // e.g this = obj
    obj.get()

Now, the `this` keyword is JavaScripts biggest gotcha. It depends on the object and not where the function was defined which causes a lot of problems and confusions. Look at the below given example:

    function getVal() { 
      return this.val; 
    }
    
    var obj1 = {
      get: getVal,
      val: "Hello, World!"
    };
    
    var obj2 = {
      get: getVal,
      val: "Greetings!"
    };
    
    console.log(obj1.get());
    console.log(obj2.get());
    console.log(getVal());

So, you have to be really careful when you are using `this`. You should always code in strict mode. You can also use `call`, `bind` and `apply` to avoid the confusion.

    // Call, apply and bind
    function getName(surname) { 
      return this.name + ' ' + surname; 
    }
    
    var person = {
      name: 'Rahul'
    };
    
    const getName2 = getName.bind(person)
    const getName3 = getName.bind(person, "Bind and pass parameter")
    
    console.log(getName.call(person, "Call"));
    console.log(getName.apply(person, ["Apply"]));
    console.log(getName2("Bind first and pass parameter later"));
    console.log(getName3());


### Prototypal Inheritance

Now, the basics has been covered. Let's pick up the common interview question.

It'd be pretty rare that you would be defining all your objects from scratch. You would be following some repetitive pattern. See the code below:

    function getVal() { 
      return this.val; 
    }
    
    // This would be a maintenance nightmare
    var obj1 = {
      get: getVal,
      val: "Hello, World!"
    };
    
    var obj2 = {
      get: getVal,
      val: "Greetings!"
    };
    
    var obj3 = {
      get: getVal,
      val: "Greetings again!"
    };
    
    console.log(obj1.get());
    console.log(obj2.get());
    console.log(getVal());

Above code would be a maintenance nightmare.

So, how can we solve it? 
Create a single object and let other objects extend from this object using `Object.create`.

    function getVal() { 
      return this.val; 
    }
    
    // This would be a maintenance nightmare
    var parent = {
      get: function() { 
        return this.val; 
      },
      val: "Hello, World!"
    };
    
    var child = Object.create(parent);
    child.val = "Hello";
    
    var grandChild = Object.create(child);
    console.log(grandChild.get()); // "Hello" retreived from child's prototype

When you call `parent.get()`, JavaScript checks for the `get` function name (which as discussed earlier is a object with name, length and prototype). It checks for `this.value`. `this` points to parent and hence, we get the `value: 'Hello, world!'` from parent.

 Now, try calling `grandChild.get()`. You will get `"Hello"`.

How?
Because JS looked in the `grandChild` object but didn't find the value or function. So, it went up the prototype chain and fetched it from the child. It'll continue going upwards until it finds the property if it exists.

If any property isn't present in the existing object, JavaScript will start fetching it from the prototype chain ([[ Prototype ]]).

`grandChild[[Prototype]].get -> Child[[Prototype]].get -> Parent.get -> "get"`

This is the fundamental of JavaScript. This is how prototypal inheritance in JavaScript works. JavaScript doesn't have any other form of ineritance other than what I am showing you here.

Every object has a prototype. Objects have `object.prototype` and functions have `functions.prototype`. `Call`, `apply` and `bind` comes from this prototype.


### Polymorphism & Method overriding

Sometimes, you want the children to behave differently from parent even when the same method name is called. This is called Polymorphism - same name, different behavior.

    Consider the following example:
    
    var parent = {
      get: function() { 
        return this.val; 
      },
      val: "Hello, World!"
    };
    
    var child = Object.create(parent);
    
    // The logic got duplicated (this.val) and mentioned, later on it might be difficult to track where this is exactly pointing to
    // So, we can call it the simplest fix but it's not the best
    child.get = function() { 
        return this.val + " modified"; 
    };
    
    console.log(parent.get());
    console.log(child.get());

You might ask me to do something like this:

    var parent = {
      get: function() { 
        return this.val; 
      },
      val: "Hello, World!"
    };
    
    var child = Object.create(parent);
    child.val = "Child says hello"; // value has been reassigned
    
    // You might ask me to do something like this
    // this gives the wrong answer. Notice that the child's value has been reassigned. We want that value
    child.get = function() { 
        return parent.get() + " modified"; 
    };
    
    console.log(parent.get());
    console.log(child.get());

The `child.get` should get the value from the child object. Instead, it get's the value from the parent object coz we called `parent.get()`.

How do we solve it?

    child.get = function() { 
        return parent.get.call(this) + " modified"; 
    };

### Classes & Instantiation

You can organize your objects however you like it. But, a known practice that has helped developers a lot is to separate the methods from data.

Look at the below example:

    // Parent is the prototype. The prototypes are known as classes.
    var parent = {
      get: function() { 
        return this.val; 
      },
      val: "Hello, World!"
    };
    
    // Child becomes the prototype. But, since it's extending to the existing class, it's called a subclass
    var child = Object.create(parent);
    child.val = "Child says hello";
    
    
    child.get = function() { 
        return parent.get.call(this) + " modified"; 
    };
    
    // Children prototypes that do not extend further from it's parent class and sub class are known instances
    grandChild1 = Object.create(child);
    child.val = "Grand Child says hello";
    
    grandChild2 = Object.create(child);
    
    
    console.log(parent.get());
    console.log(grandChild1.get());

Form the above example, what we understand is:
- Parent is the prototype. The prototypes are known as classes.
- Child becomes the prototype. But, since it's extending to the existing class/prototype, it's called a subclass
- Children prototypes that do not extend further from it's parent class and sub class are known instances

How can we decide what should be the instance or prototype?
Instances are all about **data** and the prototype is all about methods.

But, as we can see `val` is getting duplicated in all the class and instances. It violates the rule of encapsulation (the way we store data so that other developers don't have to worry about overriding it. They can simply access it) and we are duplicating the existing logic causing a big maintenance problem. 

To solve this problem, we use constructors. Look at the below given example:

    // _ is a notation to let other developers to know that it's an encapsulated value
    var parent = {
      constructor: function fn0(val) {
        this._value = val;
      },
      get: function() { 
        return this._value; 
      }
    };
    
    // Child becomes the prototype. But, since it's extending to the existing class, it's called a subclass
    var child = Object.create(parent);
    child.constructor("Child says hello");
    
    
    child.get = function() { 
        return parent.get.call(this) + " modified"; 
    };
    
    // Children prototypes that do not extend further from it's parent class and sub class are known instances
    grandChild1 = Object.create(child);
    grandChild1.constructor("Grand Child says hello");
    
    grandChild2 = Object.create(child);
    
    
    console.log(child.get());
    console.log(grandChild1.get());


### The Classical Model

In the prototypal model, we instantiated the object by creating the object and then running constructor.

It's such a common approach that JavaScript has a keyword to exactly do what we did. Before we get into classical model, let's understand prototype which I said we'll discuss later.

Whenever you create a funtion, you create 2 objects:
- Function object (name, length and prototype)
- Prototype object which has constructor pointing to the same function. Weird? I know. Try to `console.log` a function and check it's constructor. It has a empty function pointing to the same function. Not every function is supposed to be used as a class. So, you might never make use of this constructor.

How do we distinguish?
- Writre a function with the first letter in uppercase to show that you'd be creating the class.
- Write a function with the first letter with lowercase to denote it's a normal function.

Now, once again check how we created the contructor in prototypal model. Now, look at the below given code:



### Instanceof

### Future Directions

### The Definitive Guide

### Recommendations