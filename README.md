# ts-decorators

[LinkedIn - Oğuzhan Kuşlar](https://www.linkedin.com/in/oguzhankuslar/)

Decorators are a feature which can be very useful for meta-programming.

Meta-Programming means that you typically won't use decorators that often to have a direct impact on the end users of your page. So off the users visiting your page, but instead decorators are a particularly well suited instrument for writing code which is then easier to use by other developers.

* Class Decorator

    Make sure that you add "experimentalDecorators": true in your tsconfig.json. Otherwise you won't be able to use decorators in your project.

        class Person {
            name = "oguzhan";

            constructor() {
                console.log("Creating person object...");
            }
        }

        const p1 = new Person();
        console.log(p1);
            -output-
            Creating person object...
            > Person {name: "oguzhan"}


    A decorator is in the end just a function. A function you apply to something, for example to a class. Let's create a decorator.

        function Logger(constructor: Function) {
            console.log("Logging...");
            console.log(constructor);
        }

        @Logger
        class Person {
            name = "oguzhan";

            constructor() {
                console.log("Creating person object...");
            }
        }

        * Decorators execute when your class is defined. Not when it is instantiated. You don't need to instantiate your class at all. We could remove that code for instantiating the class, and we would still get that decorator output.
            
            -dec. output-
            Logging...
            class Person {
                 constructor() {
                    this.name = "oguzhan";
                    console.log("Creating person object...");
                }
            }

        * It's not the only way of how we can create a decorator though.


- Decorator Factories

    Besides creating a decorator like above, we can also define a decorator factory, which basically returns a decorator function, but allows us to configure it when we assign it as a decorator to something. Let's convert the function into a factory.

        function Logger(logString: string) {
            return function(constructor: Function) {
                console.log(logString);
                console.log(constructor);
            }
        }

        @Logger("LOGGING - PERSON")
        class Person {
            name = "oguzhan";

            constructor() {
                console.log("Creating person object...");
            }
        }


        * So now we can customize the values the decorator function uses   when it executes with 
          our factory function. 
          
          We're not executing the decorator function, but we're executing a function that will return such a decorator function.

          The advantages is that we can now pass in values which will be used by that inner returned decorator function.

          So, using decorator factories can give us more power and more possibilities of configuring what the decorator then does internally.


- Building Useful Decorators

    Let's say we want to render some template, which should be some HTML code, into some place in the DOM, where I find it with hookId:
        
        -HTML-
        <div id="app"></div>

        -TS-
        function WithTemplate(template: string, hookId: string) {
            return function(constructor: any) {
                const hookEl = document.getElementById(hookId);
                const p = new constructor();
                if(hookEl) {
                    hookEl.innerHTML = template;
                    hookEl.querySelector("h1")!.textContent = p.name;
                }
            }
        }

        @WithTemplate('<h1>My Person Object</h1>', 'app')
        class Person {
            name = "oguzhan";

            constructor() {
                console.log("Creating person object...");
            }
        }

        * Now we're not interested in the constructor, and to tell TypeScript that I'm not interested here, we can add underscore as a name, which basically signals the TypeScript, "Yeah, I know I get this argument, but I don't need it. I have to specify it though". So with this "_" , I tell you that I'm aware of it.

            function WithTemplate(..){
                return function(_: Function) {..}
            }


- Multiple Decorators

    You can add more than one decorator to a class, or anywhere else where you can use a decorator.

        @Logger("LOGGING")
        @WithTemplate("<h1>hello!</h1>", "app")
        class Person {
            name = "oguzhan";

            constructor() {
                console.log("Creating person object...");
            }
        }

        * They execute bottom up. The bottom-most decorator runs first, then thereafter, the decorators above it. "WithTemplate" runs first, then "Logger" executes.

          We're talking about the actual decorator functions. The decorator factories run earlier. There actually logger factory runs first, and then the template factory. But the execution of the actual decorator functions then happens bottom up.


- Property Decorators

    We'll create a new class, because we need a class for any decorator we wanna use, but we don't have to add all decorators directly to the class.

        function Log(target: any, propertyName: string | Symbol) {
            console.log("Property decorator!");
            console.log(target, propertyName);
        }

        class Product {
            @Log
            title: string;
            private _price: number;

            set price(val: number) {
                if(val > 0) {
                    this._price = val;
                } else {
                    throw new Error("Invalid price - should be positive!");
                }
            }

            constructor(t: string, p: number) {
                this.title = t;
                this._price = p;
            }

            getPriceWithTax(tax: number) {
                return this._price * (1 + tax);
            }
        }

        -output-
        Property decorator!
        {constructor: ƒ, getPriceWithTax: ƒ} 'title'
        ->prototype of our object (constructor, getPriceWithTax, set price, __proto__)

        * If you add a decorator to a property, the decorator receives two arguments. The first argument, is the target of the property. For an instance property, target would refer to the prototype of the object that was created. If we had a static property, target would refer to the constructor function instead.

          Second argument we get, is the property name simply.


- Accessor & Parameter Decorators

    Besides properties we can also add decorators to accessors. This will now receive three arguments. It will also get the target which again is either the prototype, if we're dealing with an instance accessor, or if we're dealing with a static one, it will be the constructor function, so we don't know it will be of type any. Then we'll have the name of the member we're dealing with. And we'll also have the property descriptor here.

        function Log2(target: any, name: string, descriptor: PropertyDescriptor) {
            console.log("Accessor decorator!");
            console.log(target);
            console.log(name);
            console.log(descriptor);
        }

        class Product {
            @Log
            title: string;
            private _price: number;

            @Log2
            set price(val: number) {
                if(val > 0) {
                    this._price = val;
                } else {
                    throw new Error("Invalid price - should be positive!");
                }
            }

            constructor(t: string, p: number) {
                this.title = t;
                this._price = p;
            }

            getPriceWithTax(tax: number) {
                return this._price * (1 + tax);
            }
        }

        -output-
        Accessor decorator!
        {constructor: ƒ, getPriceWithTax: ƒ}
        price
        {get: undefined, enumerable: false, configurable: true, set: ƒ}

    
    Besides properties and accessors, we also got methods and can add decorators to methods

    A method decorator also receives three arguments. "Target" again which if it's an instance method, is the prototype of the object. If it's a static method, the constructor function instead. "Name" of the method as second argument. Also the "descriptor" at the end. These are the three arguments we get here, these are the same arguments as in our accessor. So indeed we could also re-use it.

        function Log3(target: any, name: string | Symbol, descriptor: PropertyDescriptor) {
            console.log("Method decorator!");
            console.log(target);
            console.log(name);
            console.log(descriptor);
        }

        class Product {
            @Log
            title: string;
            private _price: number;

            @Log2
            set price(val: number) {
                if(val > 0) {
                    this._price = val;
                } else {
                    throw new Error("Invalid price - should be positive!");
                }
            }

            constructor(t: string, p: number) {
                this.title = t;
                this._price = p;
            }

            @Log3
            getPriceWithTax(tax: number) {
                return this._price * (1 + tax);
            }
        }

        -output-
        Method decorator!
        {constructor: ƒ, getPriceWithTax: ƒ}
        getPriceWithTax
        {writable: true, enumerable: false, configurable: true, value: ƒ}


    The last decorator we can add is to a parameter. It gets the "target" same as before. The next argument we get is the "name", and not the name of the parameter, but the name of the method in which we used this parameter. The last arguement, the different one, is now not the PropertyDescriptor, but instead this is the position of this argument, so the number of the argument. Here for example, this would be the first argument.

        function Log4(target: any, name: string | Symbol, position: number) {
            console.log("Parameter decorator!");
            console.log(target);
            console.log(name);
            console.log(position);
        }

        class Product {
            ...
            @Log3
            getPriceWithTax(@Log4 tax: number) {
                return this._price * (1 + tax);
            }
        }

        -output-
        Parameter decorator!
        {constructor: ƒ, getPriceWithTax: ƒ}
        getPriceWithTax
        0


- When Do Decorators Execute?

    First of all, they're all running without us instantiating the object "Product". Or in other words if we create a Product here with;
    
        const p1 = new Product("Book", 19);
        const p2 = new Product("Book 2", 29);

    This compiles without errors but our decorator code here doesn't run more often. So it's not the instantiation of this class that matters. All these decorators, no matter if it was a property decorator, an accessor decorator, a method decorator or a parameter decorator, they all executed when we defined the class "Product".

    These are not decorators that run at run time when you call a method or when you work with a property. This is not what they do. Instead these decorators allow you to do additional behind the scenes set up work when a class is defined.

    Back to that metaprogramming concept, that's the idea behind decorators or that's their core use case. They're not EventListeners you add to something so that when you work with a property you can run some code. You can make something like that work with decorators actually, but by tweaking and editing something behind the scenes, but the decorator itself really is just a function that executes when your classes defined.

    
- Returning and Changing a Class in a Class Decorator

    Some decorators, class decorators but also method decorators, for example, actually are also capable of returning something. And what you can return and what TypeScript is able to use, depends on which kind of decorator you're working with. Let's say we're working with a decorator that gets added to a class, and in such a decorator function, you can return a new constructor function, which will replace the old one. So which will replace the class to which you added to decorator you could say.

    So here we can return a new constructor function, or simply return a new class

        function WithTemplate(template: string, hookId: string) {
            console.log("TEMPLATE FACTORY");
            return function<T extends { new (...args: any[]): {name:string} }>(originalConstructor: T) {
                return class extends originalConstructor {
                    constructor(..._: any[]) {
                        super();
                        const hookEl = document.getElementById(hookId);
                        if(hookEl) {
                            hookEl.innerHTML = template;
                            hookEl.querySelector("h1")!.textContent = this.name;
                        }
                    }
                }
            }
        }

        * So now what I'm trying to do is, I try to replace the class, the constructor function to which we added our decorator, with a new class, with a new constructor function, where I still execute the old logic, but where I also add my own new logic, and therefore now the template should actually only be rendered to the DOM if I really instantiate my object.

        @WithTemplate('<h1>My Person Object</h1>', 'app')
        class Person {
            name = "oguzhan";

            constructor() {
                console.log("Creating person object...");
            }
        }


- Decorator Return Types

    Decorators where you can return something are the decorators you add to methods and the decorators you add to accessors. We can return something on them, and that something should be a descriptor, which allows us to change the method or change the configuration of the method.

    Example: Creating an "Autobind" Decorator

        -HTML-
        <button>Click me</button>

        * I now want to make sure that when we click this button we execute a method on an object.

        -TS-
        function Autobind(target: any, methodName: string | Symbol | number, descriptor: PropertyDescriptor) {
            const originalMethod = descriptor.value;
            const adjDescriptor: PropertyDescriptor = {
                configurable: true,
                enumerable: false,
                get: function(){
                    const boundFn = originalMethod.bind(this);
                    return boundFn;
                }
            }
            return adjDescriptor;
        }

        class Printer {
            message: "This works!";

            @Autobind
            showMessage() {
                console.log(this.message);
            }
        }

        const p = new Printer();

        const button = document.querySelector("button")!;
        button.addEventListener("click", p.showMessage);
        // button.addEventListener("click", p.showMessage.bind(p));


- Validation with Decorators

    class Course {
        title: string;
        price: number;

        constructor(t: string, p: number) {
            this.title = t;
            this.price = p;
        }
    }

    Of course now when we want to instantiate this course, we have to pass in a valid title and a valid price. But one common scenario you might encounter in some applications is that you fetch data, where you guess you have a couple of courses let's say, but you don't know for sure. Or,

    Another possible scenario, you let users enter the data and you simply want to assign that data and create a new course with the user-entered data and you assume it's right, but you are not guaranteed that it's right and therefore you want to validate the input.

        -HTML-
        <form>
            <input type="text" placeholder="Course title" id="title" />
            <input type="text" placeholder="Course price" id="price" />
            <button type="submit">Save</button>
        </form>

        -TS-
        interface ValidatorConfig {
            [property: string]: {
                [validatableProp: string]: string[] // ["required", "positive"]
            }
        }

        const registeredValidators: ValidatorConfig = {}

        function Required(target: any, propName: string) {
            registeredValidators[target.constructor.name] = {
                ...registeredValidators[target.constructor.name],
                [propName]: [...(registeredValidators[target.constructor.name]?.[propName] ?? []), "required"]
            };
        }
        function PositiveNumber(target: any, propName: string) {
            registeredValidators[target.constructor.name] = {
                ...registeredValidators[target.constructor.name],
                [propName]: [...(registeredValidators[target.constructor.name]?.[propName] ?? []), "positive"]
            };
        }
        function validate(obj: any) {
            const objValidatorConfig = registeredValidators[obj.constructor.name];
            if(!objValidatorConfig) {
                return true;
            }
            let isValid = true;
            for(const prop in objValidatorConfig) {
                for(const validator of objValidatorConfig[prop]) {
                    switch(validator) {
                        case "required":
                            isValid = isValid && !!obj[prop];
                            break;
                        case "positive":
                            isValid = isValid && obj[prop] > 0;
                            break;
                    }
                }
            }
            return isValid;
        }

        class Course {
            @Required
            title: string;
            @PositiveNumber
            price: number;

            constructor(t: string, p: number) {
                this.title = t;
                this.price = p;
            }
        }

        const courseForm = document.querySelector("form")!;
        courseForm.addEventListener("submit", event => {
            event.preventDefault();
            const title = document.getElementById("title") as HTMLInputElement;
            const price = document.getElementById("price") as HTMLInputElement;
        
            const title = titleEl.value;
            const price = +priceEl.value;
        
            const createdCourse = new Course(title, price);
            if(!validate(createdCourse)) {
                throw new Error("Invalid input, please try again!");
            }
            console.log(createdCourse);
        });

