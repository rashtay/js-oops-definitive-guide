
# JS OOPS: The Definitive Guide

This guide will help you understand how OOPS work specifically while building your site/app using JavaScript.

### Object Fundamentals

JavaScript has two categories of data types:
- Truly Primitives
	- undefined
	- null
	- boolean
	- string
	- number
	- object
- Special Objects
	- Arrays - [1, 2]
	- Functions - function() {}
	- RegExp /qiyo/

Objects are like dictionaries. They have a key value pair. The key is always a string. We can assign any datatype to this property.

**Primitives** are passed by value.

    // These two values are not connected. Any changes to num2 will not change num1
    const num1 = 1;
    const num2 = num1;

**Objects** are passed by reference

    // obj1 and obj2 are connected. They have an arrow pointing to the same memory
    const obj1 = { a: 1 };
    const obj2 = obj1;
    obj2.a = 2;
    
    console.log(obj1);
    // object.b doesn't exist and will return undefined. You can assign undefined too
    console.log(obj2.b);
    
    // Note: assigning undefined doesn't delete the property. We use delete keyword for deleting the property
    delete obj2.a;

### Functions & Methods

Functions are object in JavaScript. When you create a function (e.g function a() {}), JavaScript creates an object with 3 properties:
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

`func2` is referencing to the same function (remember the arrow to the same memory?). As we discussed earlier, we passed a reference. Hence, now you can call the function `func2` and expect the same results of `func1`

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
    console.log(getVal.call(obj1));  // Force the function to be bound by the object obj1

Javascript will set the `this` keyword to the object that you use. So, you have to be really careful when you are using `this`. You should always code in strict mode. You can also use `call`, `bind` and `apply` to avoid the confusion.

    // Call, apply and bind comes from function.prototype
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

    var parent = {
      get: function foo() { 
        return this.val; 
      },
      val: "Hello, World!"
    };
    
    var child = Object.create(parent);  // [[Prototype]] get's set to parent
    child.val = "Hello";
    console.log(child.get());  // => `this` is now set to child
    
    var grandChild = Object.create(child);
    console.log(grandChild.get()); // "Hello" retreived from child's prototype. Javascript will first try to find get() in child. If it doesn't find it, it'll go up the prototype

When you call `parent.get()`, JavaScript checks for the `get` function name (which as discussed earlier is a object with name, length and prototype). It checks for `this.value`. `this` points to parent and hence, we get the `value: 'Hello, world!'` from parent.

 Now, try calling `grandChild.get()`. You will get `"Hello"`.

How?
Because JS looked in the `grandChild` object but didn't find the value or function. So, it went up the prototype chain and fetched it from the child. It'll continue going upwards until it finds the property if it exists.

If any property isn't present in the existing object, JavaScript will start fetching it from the prototype chain ([[ Prototype ]]).

`grandChild[[Prototype]].get -> Child[[Prototype]].get -> Parent.get -> "get"`

This is the fundamental of JavaScript inheritance. This is how prototypal inheritance in JavaScript works. JavaScript doesn't have any other form of ineritance other than what I am showing you here.

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

	// Remember while calling `child.get()`, this is set to the child object and hence, .call(this) = .call(child)
    child.get = function() { 
        return parent.get.call(this) + " modified"; 
    };
    
    
Usually, Parent can be considered as ParentPrototype and we can remove value from the object. value can then be retrieved from the child.

Also, extend child and to some other instance and you can except their respective values with the "modified" keyword

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
    
    // This is a very small example. Imagine a lot is happening in parent.get() and we want the same code. Then it does make sense to use the same function
    child.get = function() { 
        return parent.get.call(this) + " modified"; 
    };
    
    // Children prototypes that do not extend further from it's parent class and sub class are known instances
    grandChild1 = Object.create(child);
    grandChild1.val = "Grand Child says hello";
    
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

You still are creating a new property but you are accessing it only using `.get()`

### The Classical Model

In the prototypal model, we instantiated the object by creating the object and then running constructor.

It's such a common approach that JavaScript has a keyword to exactly do what we did. Before we get into classical model, let's understand prototype which I said we'll discuss later.

Whenever you create a funtion, you create 2 objects:
- Function object (name, length and prototype)
- Prototype object which has constructor pointing to the same function. Weird? I know. Try to `console.log` a function and check it's constructor. It has a empty function pointing to the same function. Not every function is supposed to be used as a class. So, you might never make use of this constructor.

How do we distinguish?
- Write a function with the first letter in uppercase to show that you'd be creating the class.
- Write a function with the first letter with lowercase to denote it's a normal function.

Now, once again check how we created the contructor in prototypal model and then check how can we implement the classcial model. Now, look at the below given code:

    // Protypal Model
    var ParentPrototype = {
      constructor: function fn0(value) {
        this._value = value;
      },
      get: function() {
        return this._value;
      }
    }
    
    var child = Object.create(Parent);
    child.constructor(10);
    child.get(); //10
    
    
    
    // Classical Model
    function Parent(value) {
      this._value = value;
    }
    
    Parent.prototype.get = function() {
      return this._value;
    }
    
    var child = new Parent(10);
    child.get(); //10

Let's add a subclass which is a bit complex:

    // Classical Model
    function Parent(value) {
      this._value = value;
    }
    
    Parent.prototype.get = function() {
      return this._value;
    }
    
    var child = new Parent(10);
    child.get(); //10
    
    
    // Adding a subclass
    function GrandChild(value) {
      // this is wrong as GrandChild doesn't have anything in prototype. It doesn't have access to get of Parent
      // Remember how we were extending to other prototypes before overriding functions?
      // We fix it by assigning the prototype of Parent to GrandChild
      // This is just setting the value
      Parent.call(this, value)
    }
    
    GrandChild.prototype = Object.create(Parent.prototype); // Grandchild.prototype = Parent.prototype. Note that grandchild and parent were objects and now they are function. Hence, we use their prototype 
    
    // This you can do it or skip it. Basically, this makes sure our constructor is pointing to the right thing
    GrandChild.prototype.constructor = GrandChild;
    
    GrandChild.prototype.get = function() {
      return Parent.prototype.get.call(this) + "modified";
    }
    
    var finalChild = new GrandChild(20);
    finalChild.get();

### Instanceof

`instanceof` will let you know whether instance is extended from a class or not.

`child instanceof Parent; // true`
`child instanceof GrandChild; // false`
`finalChild instanceof GrandChild; // true`

### Classes

	class Rectangle {
	  constructor(height, width) {
	    this.height = height;
	    this.width = width;
	  }
	  // Getter
	  get area() {
	    return this.calcArea();
	  }
	  // Method
	  calcArea() {
	    return this.height * this.width;
	  }
	}

	const square = new Rectangle(10, 10);

	console.log(square.area); // 100
	
Now, since we have seen an example, the above code can be converted to:

    class Parent {
	  constructor(value) {
	    this._value = value;
	  }

	  getValue() {
	    return this._value;
	  }
	}

	const childInstance = new Parent(42);
	console.log(childInstance.getValue());

	class Child extends Parent {
	  constructor(value) {
	    super(value);
	  }

	  getValue() {
	    return super.getValue() + " modified!";
	  }
	}

	const grandChild = new Child(20);

	console.log(grandChild.getValue());

	const grandChild2 = new Child(40);
	console.log(grandChild2.getValue());


### Future Directions

1. Use ESLint with strict mode
2. Use ES6 classes
3. You woould be using Classical model
