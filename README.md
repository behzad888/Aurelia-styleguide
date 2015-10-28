# Aurelia Style Guide

The purpose of this style guide is to provide guidance on building Aurelia applications by showing the conventions that I use and, more importantly.

##About the this style guide
While this guide explains the what, why and how, I find it helpful to see them in practice. This guide is accompanied by a sample application that follows these styles and patterns.

##Table of Content
 1. [Startup And Configuration](#startup-and-configuration)
 2. [Views and ViewModels](#)
 3. [Templating](#)
 4. [Routing](#)
 5. [Extending HTML](#)
 6. [Eventing](#)
 7. [HTTP Client](#)
 8. [Debugging](#)
 9. [Customization](#)
 10. [JSHint](#)
 11. [Naming](#)
 12. [Animations](#)
 13. [Compiler Options](#)
 14. [Doc Generated](#)
 15. [Testing](#)

##Startup And Configuration
Most platforms have a "main" or entry point for code execution. Aurelia is no different. If you've read the Get Started page, then you've seen the aurelia-app attribute. Simply place this on an HTML element and Aurelia's bootstrapper will load an app.js and app.html, databind them together and inject them into the DOM element on which you placed that attribute.


**[Back to top](#table-of-content)**
