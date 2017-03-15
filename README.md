# Aurelia Style Guide

The purpose of this style guide is to provide guidance on building Aurelia applications by showing the conventions that I use and, more importantly.

## About the this style guide
This guide explains the *what* , *why* and *how* to see them in practice. This guide is accompanied by a sample application that follows these styles and patterns.

## Table of Content
 1. [Single Responsibility](#single-responsibility)
 2. [Application Bootstrap Config](#application-bootstrap-config)
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
 - [Contributing](#contributing)

## Single Responsibility

### Rule of 1
###### [Style [A001](#style-a001)]

If you use Typescript language follow [Typescript guideline](https://github.com/Microsoft/TypeScript/wiki/Coding-guidelines) please.

- Define 1 component per file, recommended to be less than 400 lines of code.

  *Why?*: One component per file promotes easier unit testing and mocking.

  *Why?*: One component per file makes it far easier to read, maintain, and avoid collisions with teams in source control.

  *Why?*: One component per file avoids hidden bugs that often arise when combining components in a file where they may share variables, create unwanted closures, or unwanted coupling with dependencies.

**[Back to top](#table-of-content)**

### Small Functions
###### [Style [A002](#style-a002)]

 - Define small functions, no more than 75 LOC (less is better).

  *Why?*: Small functions are easier to test, especially when they do one thing and serve one purpose.

  *Why?*: Small functions promote reuse.

  *Why?*: Small functions are easier to read.

  *Why?*: Small functions are easier to maintain.

  *Why?*: Small functions help avoid hidden bugs that come with large functions that share variables with external scope, create unwanted closures, or unwanted coupling with dependencies.

**[Back to top](#table-of-content)**

## Application Bootstrap Config
Aurelia was originally designed for Evergreen Browsers. This includes Chrome, Firefox, IE11 and Safari 8. However, we have identified how to support IE9 and above. To make this work, you need to add an additional polyfill for MutationObservers. This can be achieved by a jspm install of `github:polymer/mutationobservers`. Then wrap the call to `aurelia-bootstrapper` as follows:

```html
<!-- recommended -->
<!-- Based on Aurelia Hub -->

<!doctype html>
<html>
  <head>
    <title>My App</title>
  </head>
  <body>
    <script src="jspm_packages/system.js"></script>
    <script src="config.js"></script>
    <script>
      SystemJS.import('raf/polyfill').then(function() {
        return SystemJS.import('aurelia-polyfills');
      }).then(function() {
        return SystemJS.import('webcomponents/webcomponentsjs/MutationObserver');
      }).then(function() {
        SystemJS.import('aurelia-bootstrapper');
      });
    </script>
  </body>
</html>
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

In case you use Webpack, you can replace the `aurelia-bootstrapper-webpack` package with the `./src/main` entry file in the `aurelia-bootstrapper` bundle defined inside of `webpack.config.js`, and call the bootstrapper manually:
```javascript
/* recommended */
/* Based on Aurelia Hub */

import {bootstrap} from 'aurelia-bootstrapper-webpack';

bootstrap(async aurelia => {
  aurelia.use
    .standardConfiguration()
    .developmentLogging();

  await aurelia.start();
  aurelia.setRoot('app', document.body);
});
```

**[Back to top](#table-of-content)**

## Views and ViewModels
In Aurelia, user interface elements are composed of view and view-model pairs. The view is written with HTML and is rendered into the DOM. The view-model is written with JavaScript and provides data and behavior to the view. The templating engine and/or DI are responsible for creating these pairs and enforcing a predictable lifecycle for the process. Once instantiated, Aurelia's powerful databinding links the two pieces together allowing changes in your data to be reflected in the view and vice versa. This Separation of Concerns is great for developer/designer collaboration, maintainability, architectural flexibility, and even source control.

### Dependency Injection

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
/* recommended */

/* App.js */
import {inject} from 'aurelia-framework'; // avoid from this line
import {HttpClient} from 'aurelia-fetch-client';

export class App {
 static inject = [HttpClient];
  constructor(http) {
   this.http = http;
  }

```
ES2016:
```javascript
/* recommended */

/* App.js */
import {inject} from 'aurelia-framework'; // avoid from this line
import {HttpClient} from 'aurelia-fetch-client';

export class App {
  static get parameters() {
    return [[HttpClient]];
  }
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
export class App {
    constructor(private http:HttpClient) {
        this.http = http;
    }
}
```

### Resolver
When explicitly declaring dependencies, it's important to know that they don't have to be just constructor types. They can also be instances of `resolvers`
```javascript
/* recommended  */

/* app.js  */
import {Lazy, inject} from 'aurelia-framework';
import {HttpClient} from 'aurelia-fetch-client';

@inject(Lazy.of(HttpClient))
export class App{	  
	constructor(getHTTP){
		this.http = getHTTP;
	}
}
```
For `static` use this:
```javascript
/* recommended  */

/* app.js  */
import {Lazy} from 'aurelia-framework';
import {HttpClient} from 'aurelia-fetch-client';

export class App {	  
 static inject = [Lazy.of(HttpClient)]
	constructor(getHTTP) {
		this.http = getHTTP;
	}
}
```


**[Back to top](#table-of-content)**


### Contributing
Open an issue first to discuss potential changes/additions. If you have questions with the guide, feel free to leave them as issues in the repository. If you find a typo, create a pull request. The idea is to keep the content up to date and use github’s native feature to help tell the story with issues and PR’s, which are all searchable via google. Why? Because odds are if you have a question, someone else does too! You can learn more here at about how to contribute.
**[Back to top](#table-of-content)**
