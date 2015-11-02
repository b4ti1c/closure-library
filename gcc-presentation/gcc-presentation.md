# GOOGLE CLOSURE COMPILER
## Taming JS & Programming Object-Oriented 


## What is GCC

- Google tools of compiling JS projects, for real
- Foundations for Object-Oriented Programming
- Self-documentation & Typechecking
- Very Aggressive Minification/Uglification
- Event-Driven Programming
- Namespace Management
- Cross-browser compatible libraries
- Syntax control
- Access-Control
- Somehow enforces JS styleguide


## Hardcore Minification

Normally, minification just removes the whitespace

Normally, uglification just replaces the name of the variables, class names and properties to short strings like `a1, a2, b, C` etc..

Normally, you would need to either concatanate all your code or use a mapping file for uglification to work on multiple js files.

Normally, this radically compresses our JS files so that they are loaded much faster to the server.

But there is better.

##### Uglification by Closure Compiler

Closure Compiler performs all of those above

**PLUS**, it makes radical changes in your code such that you won't event understand or recognize. How come?

It analyzes your code, your calls, your semantics and replaces them with the fastest option it can come up with. 

This results with

- Event better Compression
	- In my case, gcc compresses %50 better than UglifyJs with mangling enabled. (230kB -> GCC: 14kB, Uglify: 27kB)
- Slightly greater performance (code-wise), e.g. lesser function calls, reduction in #of scopes, #of parameters of methods ...


What is `O(J, this.Oc)` here? GCC must have changed the definition of the function call

![Obfuscation](http://tardis1.tinygrab.com/grabs/af1a6757d98d621d0cf9ea748b33cd1ac307b1f66f.png)

Another Example

![Obfuscation2](http://tardis1.tinygrab.com/grabs/af1a6757d9dd1a16ae7721e80a6bcd9fbd69b94fb5.png)


Our Goal:

- **Very Readable Codebase** with lots of documentation
- **Very Obfuscated Compilation** such that an observer in client won't understand anything at all.


## Object-Oriented Javascript

We should always write code as if the `'use strict'` command was there. Following the [google's js styling guide](https://google.github.io/styleguide/javascriptguide.xml) ensures that. It will also cosmetically beautify your code. Go ahead and have a quick read at it. You should **never** go wild while writing JS in order to maintain you and your peers' sanity.


### Module Manager

Similar to NodeJS' `module.export` and `require`

- goog.provide, goog.require

- goog.inherits

- goog.addSingletonGetter, object.getInstance

##### Object Definition

```js
	goog.provide('app.components.Button');



	app.component.Button = function(){
		this.click = ... //some action
	};
```

##### Object Inheritance

```js
	goog.provide('app.components.LinkButton');
	goog.require('app.components.Button');



	app.component.Button = function(){
		goog.base(this);

		this.goToLink = 'someUrl'
		...
	};
	goog.inherits(app.components.LinkButton, app.components.Button);
```

### Singletons

These guys are objects that live throughout the app life-time. They are instantiated only once.

- goog.addSingletonGetter

```js
	goog.provide('app.managers.ComponentManager');



	app.managers.ComponentManager = function(){
		this.componentCount = 0;
		...	
	};
	goog.addSingletonGetter(app.managers.ComponentManager);
```

For example later on in another document:

```js
    console.log(app.managers.ComponentManager.getInstance().componentCount)
```

## Self Documentation & TypeChecking

The code above will not work if you do not document it properly.

Closure Compiler will be complaining about a constructor being called that is not explicitly annotated.

### Google Annotations

Read [official docs](https://developers.google.com/closure/compiler/docs/js-for-compiler?hl=en) if you get stuck with a warning/error

**Our goal** is **0 warnings, 0 errors** during compilation


##### Constructors and Inheritance

- goog.inherits
- @constructor
- @extends

Constructors

```js
	goog.provide('app.components.Button');



	/**
	 * @constructor
	 */
	app.component.Button = function(){
		this.click = ... //some action
	};
```

Inherited objects
 
```js   
	goog.provide('app.components.LinkButton');
	goog.require('app.components.Button');


	/**
	 * @constructor
	 * @extends {app.components.Button}
	 */
	app.component.Button = function(){
		goog.base(this);

		this.goToLink = 'someUrl'
		...
	};
	goog.inherits(app.components.LinkButton, app.components.Button);
```

##### Typechecking of Function Parameters

- @param
- @return

```js
	/**
	 * @constructor
	 * @extends {app.components.Button}
	 */
	app.component.Button = function(){
		goog.base(this);

		this.goToLink = 'someUrl'
		...
	};
	goog.inherits(app.components.LinkButton, app.components.Button);


	/**
	 * This function overwrites the link that the click function calls; 
	 *
	 * @param {string} newLink New link that will replace the current 
	 */
	app.component.Button.prototype.changeGoToLink = function(newLink){
		this.goToLink = newLink;
	};
```

Are parameters optional? `changeGoToLink()`

    * @param {string=} newLink

Are parameters nullable? `changeGoToLink(null)`
	
	* @param {?string} newLink

Is it both?

	* @param {?string=} newLink

Common types

	* @param {Object}
	* @param {Array}
	* @param {Function}
	* @param {string}
	* @param {number}
	* @param {boolean}

Exclusive types

	* @param {app.components.Button}

Very specific annotations

	* @param {function(number, string)}

Parameter can be different types?

	* @param {string|number|Object}

Parameter can be anything?

	* @param {*}

### Strict Typechecking

- Closure compiler will **NOT** compile, and will raise an error if you call `changeGoToLink(true)`
- Because we are passing a boolean parameter in the place of a string.
- You cannot do `changeGoToLinkl()` as well if you did not have it nullable, that is: `* @param {string=} ...`
- You can even set the return types of functions so that function always returns the desired types. `* return {string}`

### Access Control

Implementation of private & protected variables

- Private properties can only be accessed from the same js file.
	- `* @private`
- Protected properties can be accessed from the same js file and from subclasses only.
	- `* @protected`
	
### Event-Driven Programming

- goog.events.EventTarget
- setParentEventTarget
- dispatchEvent
- goog.events.listen, goog.events.listenOnce

All objects inherit from goog.events.EventTarget in an application

Events bubble from child to parent.

	//Assume object `x` inherits from goog.events.EventTarget

parent explicitly sets his children

```js
	x.setParentEventTarget(parentObject);
```

or

```js
	parentObject.x.setParentEventTarget(this);
```

child fires events

- `x.dispatchEvent('eventName')`
- `x.dispatchEvent({name: 'eventName', data1: ... , data2: ...})`

anybody can listen 

- `goog.events.listen(listeningContext, eventName, eventCallback, eventPreference, callbackContext)`
- `goog.events.listenOnce(...)`

### Namespace Management

We will (almost) never clutter the global namespace

	app.Bootstrapper
	app.managers.XManager
	app.managers.YManager
	app.services.AService
	app.agents.CudaAgent
	app.components.Component1
	app.components.Component2
	app.views.View1
	...

### Cross-Browser Compatible Libraries

If goog library has a method for it, go with it.

Google implemented & re-implemented many utilities, and using them ensures that your code will behave **exactly the same** on all browsers.

Google Closure Library is very well documented. Use their [git-history docs](https://closure-library.googlecode.com/git-history/docs/), or google it directly.

- XMLHttpRequest -> goog.net.XhrIo
- Dom Operations -> goog.dom
	- element, node get set methods
	- add/remove classes
	- basically any js code that acts on html elements exist in goog.dom library
	- check out [goog.dom.js documentation](https://closure-library.googlecode.com/git-history/docs/local_closure_goog_dom_dom.js.html)
- goog.arrays
- goog.crypt

- There is definitely more like this...
