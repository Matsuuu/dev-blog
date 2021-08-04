---
title: "Writing JavaScript, but with types!"
date: 2021-08-04T19:50:38+02:00
---

As someone who has worked with strongly typed languages like Java and C# and loosely typed languages like JavaScript,
I've often run into a situtation in which I've wished my JavaScript code would have types and they would be enforced.
This would save me from a lot of runtime headache that can happen for example when a string variable is passed into a function
expecting a number.

The result is the dreaded situation every js developer has run into at some point of their carreer

```javascript
function addFive(num) {
    return num + 5;
}
addFive("5")
// "55"
```

Oh how it would be great if there was something to tell us we're going to be doing something silly in our editor.

```javascript
addFive("5");   Argument of type 'string' is not assignable to parameter of type 'number'.
```

Well you might be thinking:

> This is literally what [TypeScript](http://typescriptlang.org/) is for.

And I'm fully aware of it. As someone who has worked with both, there are upsides in using TypeScript, but sometimes you might
just want (or even need) to write JavaScript instead.

This however is not going to be a comparison between TypesScript and JavaScript, and therefore I will not go into comparing them.

What we _are_ going to be looking at is how you can enable this kind of type checking in your JavaScript projects with ease.


## Adding TypeScript... sort of.

We will be writing all of our code in plain JavaScript, but to get the type checking, we still require the TypeScript type checking functionalities.
To add them to our project, we need to install TypeScript as a dev dependency.

```bash
npm init -y
npm install -D typescript
```

We will also create a command to run the TypeScript compiler

```json
{
  "name": "",
  "type": "module",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
      "build": "tsc -p jsconfig.json"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "devDependencies": {
    "typescript": "^4.3.5"
  }
}
```

Now we're set to start configuring our type checking system.

## jsconfig.json

For anyone who has worked with TypeScript, a `tsconfig.json` -file is familiar. It contains the configurations on how the TypeScript compiler
should treat the project, and how strict it should be about the typings in the project.

For our case, we could create a tsconfig.json file, but instead, we are going to be creating a `jsconfig.json` -file.

[jsconfig.json](https://code.visualstudio.com/docs/languages/jsconfig) is a file supported by editors like [Visual Studio Code](https://code.visualstudio.com/docs/languages/jsconfig)
and is used to mark the directory as a root of a JavaScript Project.

This also has the added benefit of having the JavaScript-related compiler flags enabled by default.

Our minimal jsconfig -file is going to look as follows:

```json
{
  "compilerOptions": {
    "module": "esnext", // Specifies the module system, when generating module code.
    "target": "esnext", // Specifies which default library (lib.d.ts) to use
    "checkJs": true, // Enable type checking on JavaScript files.
    "moduleResolution": "node", // Specifies how modules are resolved for imports

    "declaration": true, // Generate .d.ts files for every TypeScript or JavaScript file inside your project
    "outDir": "dist", // If specified, .js (as well as .d.ts, .js.map, etc.) files will be emitted into this directory
    "noEmit": false, // Do not emit compiler output files like JavaScript source code, source-maps or declarations.

    "allowJs": true, // Allow JavaScript files to be imported inside your project, instead of just .ts and .tsx files
  },
  "exclude": ["node_modules", "dist"]
}
```

Next we create a JavaScript -file, and try to build it:

```javascript
// index.js
function addFive(num) {
    return num + 5;
}
const ten = addFive("5")
console.log(ten);
```

If we run `npm run build` and then run `node dist/index.js`, we will be greeted with the dreaded string concatenation

```bash
$: npm run build

> typing-testing@1.0.0 build
> tsc -p jsconfig.json

$: node dist/index.js
55
```

Now let's start typing out our JavaScript code!


### JSDoc

Since in JavaScript, we don't have the same syntax as in TypeScript for declaring types, we will instead do it via [JSDoc](https://jsdoc.app/).

What would look like this in TypeScript...

```typescript
function addFive(num: number) {
    return num + 5;
}
const ten = addFive("5")
console.log(ten);
```

Will look like this in JavaScript:

```javascript
/**
 * @param { number } num
 **/
function addFive(num) {
    return num + 5;
}
const ten = addFive("5")
console.log(ten);
```

And if we now try to build our project...

```bash
$: npm run build

> typing-testing@1.0.0 build
> tsc -p jsconfig.json

index.js:8:21 - error TS2345: Argument of type 'string' is not assignable to parameter of type 'number'.

8 const ten = addFive("5");
                      ~~~


Found 1 error.

```

Voilà! We have type checking functioning in our JavaScript code! 

This of course is only in "build" time, since it's JavaScript after all, but this is already way better than working without any types at all.

### Editors

A lot of the major browsers can read jsconfig.json and JSDoc, therefore editors like VS Code are able to also provide these type hints 
in-editor. So on top of catching these type errors on build time, we also have our editor informing us about them.

### Bonus

As a nice bonus cool trick I'll show you something that was only a daydream of mine until I realised it was possible:

**Decorators in JavaScript**

As someone who works a lot with Java, I _love_ decorators. They make it so easy to annotate your code with little snippets of functionality
without having to write function calls or anything of sorts.

In TypeScript, developers are able to use decorators to their heart's content, but at least for now, it's not a built in JavaScript functionality.

But if you've payed attention, you might realize that we _are_ working with the TypeScript compiler here.

So let's get into trying out decorators!

---

First up, we need to enable decorators in our `jsconfig.json`.

```json
{
  "compilerOptions": {
    "module": "esnext", // Specifies the module system, when generating module code.
    "target": "esnext", // Specifies which default library (lib.d.ts) to use
    "checkJs": true, // Enable type checking on JavaScript files.
    "moduleResolution": "node", // Specifies how modules are resolved for imports

    "declaration": true, // Generate .d.ts files for every TypeScript or JavaScript file inside your project
    "outDir": "dist", // If specified, .js (as well as .d.ts, .js.map, etc.) files will be emitted into this directory
    "noEmit": false, // Do not emit compiler output files like JavaScript source code, source-maps or declarations.

    "allowJs": true, // Allow JavaScript files to be imported inside your project, instead of just .ts and .tsx files
    "experimentalDecorators": true // Enables experimental support for decorators
  },
  "exclude": ["node_modules", "dist"]
}
```

Now we should be able to write some decorators!

```javascript
function logClass(clazz) {
    console.log(clazz.name);
}

@logClass
class Foo {
    constructor() {
        console.log("Foo Constructor");
    }
}

new Foo();
```

If we now build and run our project, we should have the class' name logged to our console.

```bash
$: npm run build

> typing-testing@1.0.0 build
> tsc -p jsconfig.json

$: node dist/index.js
Foo
Foo Constructor
```

Awesome! We have decorators working with our JavaScript code!

### Ending words

While TypeScript has a lot of features that just cannot be achieved with plain JavaScript, the type system can be 
utilized to a certain point with JavaScript too, and it can provide extremely useful when working with complicated data and
stricly typed functions.

But for example in cases where you're working with projects, where you want to ship your code "as is", but also want to have 
a robust type system, this approach can serve you extremely well.

I've used the JSDoc + JavaScript typing system in my latest projects, and it's been extremely helpful, since I am able to 
clearly describe my functions and API's, and have the editor immediately notify me, when something is wrong.

Another upside of this system is that you can unplug it when you want to. With TypeScript, it isn't such a simple task to for example
just ignore the type system completely.

---

Links to study more

- [TypeScript Documentation - tsconfig/jsconfig](https://www.typescriptlang.org/docs/handbook/tsconfig-json.html)
- [TypeScript Documentation - tsconfig/jsconfig reference](https://www.typescriptlang.org/tsconfig)
- [JSDoc official docs site](https://jsdoc.app/)

