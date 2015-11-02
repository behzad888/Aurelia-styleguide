# Aurelia Style Guide

The purpose of this style guide is to provide guidance on building Aurelia applications by showing the conventions that I use and, more importantly.

##About the this style guide
This guide explains the *what* , *why* and *how* to see them in practice. This guide is accompanied by a sample application that follows these styles and patterns.

##Table of Content
 1. [Application Bootstrap Config](#application-bootstrap-config)
 2. [Single Responsibility](#single-responsibility)
 3. [Views and ViewModels](#views-and-viewmodels)
 - [Templating](#)
 - [Routing](#)
 - [Extending HTML](#)
 - [Eventing](#)
 - [HTTP Client](#)
 - [Debugging](#)
 - [Customization](#)
 - [JSHint](#)
 - [Naming](#)
 - [Animations](#)
 - [Compiler Options](#)
 - [Doc Generated](#)
 - [Testing](#)

##Application Bootstrap Config
Aurelia was originally designed for Evergreen Browsers. This includes Chrome, Firefox, IE11 and Safari 8. However, we have identified how to support IE9 and above. To make this work, you need to add an additional polyfill for MutationObservers. This can be achieved by a jspm install of `github:polymer/mutationobservers`. Then wrap the call to `aurelia-bootstrapper` as follows:

```html
<!-- recommended -->

<script src="jspm_packages/system.js"></script>
<script src="config.js"></script>
<script>
  // Loads WeakMap polyfill needed by MutationObservers
  System.import('core-js').then(function() {
    // Imports MutationObserver polyfill
    return System.import('polymer/mutationobservers');
  }).then(function() {
    // Ensures start of Aurelia when all required IE9 dependencies are loaded
    System.import('aurelia-bootstrapper');
  });
</script>
```

Let's give our app the name `sample-app` and reflect this in the body tag of `index.html`
```html
<!-- recommended -->

<body aurelia-app="sample-app">
    <script src="jspm_packages/system.js"></script>
    <script src="config.js"></script>
    <script>
      System.config({
        "paths": {
          "*": "src/*.js"
        }
      });
    </script>
    <script>
      System.import('aurelia-bootstrapper');
    </script>
  </body>
```
Aurelia looks for a JavaScript file with the same name in the `src` directory for the main app config details.

**[Back to top](#table-of-content)**

##Single Responsibility
Most platforms have a "main" or entry point for code execution. Aurelia is no different. If you've read the Get Started page, then you've seen the aurelia-app attribute. Simply place this on an HTML element and Aurelia's bootstrapper will load an app.js and app.html, databind them together and inject them into the DOM element on which you placed that attribute.

The folowing example defines the `app`

```javascript
/**
* App.js
*/
export class App {
  constructor() {
   this.message = "";
  }
  
  activate() {
   this.message = "Hello, World!";
  }
  
  changeMessage(){
   this.message = "Goodbye!";
  }
}

```
Once Aurelia finds and activates app.js, the framework will try to load an app.html file to function as the view for the model. We’ll create app.html

```html
<template>
    <div>
        <div>${message}</div>
        <button click.trigger="changeMessage()">Say Goodbye</button>
    </div>
</template>

```

**[Back to top](#table-of-content)**

##Views and ViewModels
In Aurelia, user interface elements are composed of view and view-model pairs. The view is written with HTML and is rendered into the DOM. The view-model is written with JavaScript and provides data and behavior to the view. The templating engine and/or DI are responsible for creating these pairs and enforcing a predictable lifecycle for the process. Once instantiated, Aurelia's powerful databinding links the two pieces together allowing changes in your data to be reflected in the view and vice versa. This Separation of Concerns is great for developer/designer collaboration, maintainability, architectural flexibility, and even source control.

###Dependency Injection

let's define a typical view-model class
```javascript

/* recommended  */

/* App.js */
import {inject} from 'aurelia-framework';
import {HttpClient} from 'aurelia-fetch-client';

@inject(HttpClient)
export class App {
  constructor(http) {
   this.http = http;
  }

```
For static inject use this 
```javascript
/* recommended  */

/* App.js */
import {HttpClient} from 'aurelia-fetch-client';


export class App {
  static inject = [HttpClient];
  constructor(http) {
    this.http = http
  }

```
```javascript
/* avoid */

/* App.js */
import {inject} from 'aurelia-framework'; // avoid from this line
import {HttpClient} from 'aurelia-fetch-client';

export class App {
 static inject = [HttpClient];
  constructor(http) {
   this.http = http;
  }

```
```javascript
/* avoid */

/* App.js */
import {inject} from 'aurelia-framework';
import {HttpClient} from 'aurelia-fetch-client';

export class App {
  constructor() {
   this.http = new HttpClient();
  }

```
If you are working with TypeScript, you can use the --emitDecoratorMetadata compiler flag along with Aurelia's @autoinject decorator to enable the framework to read the standard TS type information. As a result, there's no need to duplicate the types. Here's what that looks like:
```javascript
/* recommended  */

/* App.ts */
import {autoinject} from 'aurelia-framework';
import {HttpClient} from 'aurelia-fetch-client';

@autoinject
export class CustomerDetail {
    constructor(private http:HttpClient) {
        this.http = http;
    }
}
```

###Resolver
When explicitly declaring dependencies, it's important to know that they don't have to be just constructor types. They can also be instances of `resolvers`

**[Back to top](#table-of-content)**
