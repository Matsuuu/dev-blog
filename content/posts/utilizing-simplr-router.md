---
title: "Utilizing Simplr Router"
date: 2021-01-12T10:52:38+02:00
---

I started working on [Simplr Router](https://github.com/Simplr/simplr-router) in September 2019. Back then I had just learned about Web Components
and the power they enable for developers. We were working on our first bigger project and were using [LitElement](https://lit-element.polymer-project.org/).
We had great tooling, but were missing one thing: A router which suited our needs and customizability.

That's when I started working on Simplr Router.

---

## The Dream

From the beginning, I have wanted to keep Simplr Router at 0 dependecies, and so far we've been able to
keep it that way. The idea of adding a router to your project, without

1. Adding multiple dependencies with it
2. Adding kilobytes upon kilobytes of arbitrary code to your project

was something I really enjoyed.

The design idea going forward with the router is to keep it as lightweight as possible, and to utilize
the latest features of the Web.

Currently Simplr Router is powered by Web Components and therefore they are the main environment for it too.

I didn't want to restrict it to just one ecosystem though. That's why late last year, I added support for middleware.

And now there is already compatibility-packages for [React](https://github.com/Simplr/simplr-router-react-middleware)
and [Preact](https://github.com/Simplr/simplr-router-preact-middleware).

---

## Enough talking. How do I use it?

Allright that's enough about the history of Simplr Router. Now let's get into the present: How to use it!

### Installation

Installing Simplr Router is the same as any other NPM package, so first we run

```bash
npm install @simplr-wc/router
```

And after that we are ready to go! As the router is written in vanilla Javascript, we don't even need
any build tools to get up and working.

However to make development easier, I usually use the [@web/dev-server](https://modern-web.dev/docs/dev-server/overview/) by [Modern Web](https://modern-web.dev/)

Next let's write some views to navigate to!

---

### Getting started

In these examples we will go through a setup with _Vanilla Web Components_, but we'll go through a Preact example [at the end of the post](#preact-example).

First we want to create our views, into which we will be navigating to with the Router.

Our base for the pages will be the following:

**Home Page**

```javascript
class HomePage extends HTMLElement {
  constructor() {
    super();
    // Create Shadow Root
    this.attachShadow({ mode: "open" });
  }

  connectedCallback() {
    // Create Template for content
    const template = document.createElement("template");
    template.innerHTML = `
      <style>
        :host {
          width: 100%;
          height: 100%;
          background: darkgreen;
          font-size: 28px;
          color: #FFF;
          display: flex;
          align-items: center;
          justify-content: center;

        }
      </style>
      <h1>Welcome to the home page</h1>
    `;

    // Add Template content to the shadow Root of the element
    this.shadowRoot.appendChild(template.content.cloneNode(true));
  }
}

if (!customElements.get("home-page")) {
  customElements.define("home-page", HomePage);
}
```

And **Profile Page**

```javascript
class ProfilePage extends HTMLElement {
  constructor() {
    super();
    // Create Shadow Root
    this.attachShadow({ mode: "open" });
    // Initialize default value
    this.user = "World";
  }

  connectedCallback() {
    // Create Template for content
    const template = document.createElement("template");
    template.innerHTML = `
      <style>
        :host {
          width: 100%;
          height: 100%;
          background: darkblue;
          font-size: 28px;
          color: #FFF;
          display: flex;
          align-items: center;
          justify-content: center;

        }
      </style>
      <h1>Welcome to the Profile page, ${this.user}</h1>
    `;

    // Add Template content to the shadow Root of the element
    this.shadowRoot.appendChild(template.content.cloneNode(true));
  }
}

if (!customElements.get("profile-page")) {
  customElements.define("profile-page", ProfilePage);
}
```

On top of these views, we will have a navbar in just pure HTML.

```html
<nav>
  <a href="/">Home</a>
  <a href="/profile">Profile</a>
  <a href="/profile/John">John</a>
</nav>
```

You don't need to fully understand Web Components to understand this demo, but the main part you
need to understand is that our views are now HTML elements, and can be rendered into the DOM
by using `<home-page></home-page>` and `<profile-page></profile-page>`.

---

### Initializing the routes

Now we get to the fun part! We will be creating the routes for our router.

The Simplr Router uses JSON as the routing table format due to it being widely used and
easily configurable. It also allows us to ship the routing file as a separate file if we wanted to.

There is a lot of configurable parts for the routes, but we can get by with just a few JSON properties

```javascript
// Initialize a array of routes, each route being it's own JSON object
const routes = [
  {
    path: "/",
    component: "home-page",
  },
  {
    path: "profile",
    component: "profile-page",
  },
];
```

We can get by with just declaring a path for our view, and the component used to render the view.

What if we wanted to create a dynamic profile page, in which the user's name would be given as a URL parameter?

That's fully possible, and quite simple.

```javascript
// Initialize a array of routes, each route being it's own JSON object
const routes = [
  {
    path: "/",
    component: "home-page",
  },
  {
    path: "profile",
    component: "profile-page",
    // Initialize the sub routes for the profile page
    // in this case /profile/:user, where :user is replaced with
    // the user's name
    routes: [
      {
        path: ":user",
        component: "profile-page",
      },
    ],
  },
];
```

By declaring a `routes` -property in our route, we can declare sub-routes for our route.

Sub-routes inherit the base path from their parent, and can be static or dynamic, like in our example above.

The parameter from the URL is mapped onto the view, and is easily usable from within the view

Inside our view component, we can access the name of the user through the keyword `this`.

```javascript
// When navigating to /profile/John
console.log(this.user);
// prints out "John"
```

### Putting it all together

Now that we have our views, we have our navigation elements and our routes, we're ready to put it all together.

We need to import the Simplr Router, and initialize it with out routes

```javascript
// Import the web components views
import "./home.js";
import "./profile.js";
import SimplrRouter from "@simplr-wc/router";

const routes = [
  {
    path: "/",
    component: "home-page",
  },
  {
    path: "profile",
    component: "profile-page",
    routes: [
      {
        path: ":user",
        component: "profile-page",
      },
    ],
  },
];

// Create a instance of SimplrRouter. Pass routes as a JSON property
const router = new SimplrRouter({ routes });
// Initialize the router.
// The router won't start before the init command is run.
router.init();
```

![Demo](/router-demo.gif)

---

### Customizing

---

### Code Splitting

---

### Middleware

---

### Preact example
