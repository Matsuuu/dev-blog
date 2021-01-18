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
3. Introducing a whole lot of refactoring or a high learning curve

was something I really enjoyed.

#

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
any build tools to get up and running.

However to make development easier, I usually use the [@web/dev-server](https://modern-web.dev/docs/dev-server/overview/) by [Modern Web](https://modern-web.dev/)

Next let's write some views to navigate to!

---

### Getting started

In these examples we will go through a setup with _Vanilla Web Components_, but we'll show a Preact example [at the end of the post](#preact-example).

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

// Declare a custom Component for our view, so that
// we can call it as <home-page></home-page>
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

// Declare a custom Component for our view, so that
// we can call it as <profile-page></profile-page>
if (!customElements.get("profile-page")) {
  customElements.define("profile-page", ProfilePage);
}
```

On top of these views, we will have a navbar in just pure HTML prepended at the start of the index.html file.

```html
<nav>
  <a href="/">Home</a>
  <a href="/profile">Profile</a>
  <a href="/profile/John">John</a>
</nav>
```

**⚠️ You don't need to fully understand Web Components to understand this demo, but the main part you
need to understand is that our views are now HTML elements, and can be rendered into the DOM
by using `<home-page></home-page>` and `<profile-page></profile-page>`.⚠️**

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
    routes: [
      // Initialize the sub routes for the profile page
      // in this case /profile/:user, where :user is replaced with
      // the user's name
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

Inside our view component, we can access the name of the user in it's property through the keyword `this`.

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

And Horaaah! We have our router already up and running!

![Demo](/router-demo.gif)

**As you might have noticed, we didn't need to alternate our anchor tags at all to get the routing functional.**

**The router handles the events on anchors itself and doesn't require developer interferance**

Next we'll look into customizing our router a bit!

---

### Customizing

Now that we have Simplr Router up and running, we are ready to customize it a bit. Let's start by modifying the transitions

#### Removing transitions

If one wanted to remove the transitions alltogether, there's a option to do just that.

```javascript
const router = new SimplrRouter({ routes, disableTransition: true });
```

This will make the page changes instant, instead of the default sliding animation the router provides out of the box.

#### Changing transitions

If however we wanted a transition, but didn't want it to be the default, we can easily modify it.

The first modification type would be the transition speed. This can be easily set in the initial configuration:

```javascript
// Transition speed is calculated in milliseconds
const router = new SimplrRouter({ routes, transitionSpeed: 1000 });
```

If the sliding transition is not something we want, we can also animate the whole transition ourselves.

First we disable the default transition, and set the transitionSpeed to a slower speed.

```javascript
const router = new SimplrRouter({
  routes,
  disableTransition: true,
  transitionSpeed: 1000,
});
```

After that, we modify the CSS of our router's container elements. We modify the entering and leaving
elements so that the fade-in-out effect applies to both of the views.

```css
simplr-router-container[entering-view],
simplr-router-container[leaving-view] {
  opacity: 0;
}
```

And now we have a nice slow fade-in-out animation for our page.

These can be of course modified in any way you like. These are just some simple examples to get started.

![Transition demo](/router-transition.gif)

#### Custom error pages

In many cases we want to display a error page when the user navigates to a faulty path.
For these cases we can declare a `not-found` path for when a view is not found and a `forbidden` path
for when access to a certain view is denied by a guard.

##### Not Found

A not found page can be configured by creating a path named ´not-found´ and assigning it a view:

```javascript
const routes = [
  {
    path: "/",
    component: "main-page",
  },
  {
    path: "not-found",
    component: "not-found-page",
  },
];

const router = new SimplrRouter({ routes });
router.init();
```

Now every time a user navigates to a view not recognized by the router, they will be greeted by your "Not Found" -page.

##### Forbidden

A forbidden page can be configured in the same manner. A forbidden page is triggered when a
[guard](https://router.matsu.fi/recipes/guards) fails it's check.

```javascript
const routes = [
  {
    path: "/",
    component: "main-page",
  },
  {
    path: "forbidden",
    component: "forbidden-page",
  },
];

const router = new SimplrRouter({ routes });
router.init();
```

#### Custom Actions

If however you would like to run a piece of code when a invalid page is loaded, that is fully possible too.

```javascript
const routes = [
  {
    path: "/",
    component: "main-page",
  },
];

const router = new SimplrRouter({
  routes,
  notFoundAction: () => alert("Page not found"),
  forbiddenAction: () => alert("You are not allowed to view this page"),
});
router.init();
```

---

### Code Splitting

The next coolest thing Simplr Router provides is the possiblity for code splitting.

What code splitting means is that you don't have to ship a huge bundle of javascript to your user
on their initial page load, but instead you can only load the view component **when it's needed**.

The best past is: You barely have to do any work to enable this. All you have to do is
instead of importing your views at the top of your file as we've done in the examples, you will
do it inside the routes.

```javascript
import SimplrRouter from "@simplr-wc/router";

const routes = [
  {
    path: "/",
    component: "home-page",
    import: () => import("./home.js"),
  },
  {
    path: "profile",
    component: "profile-page",
    import: () => import("./profile.js"),
    routes: [
      {
        path: ":user",
        component: "profile-page",
        import: () => import("./profile.js"),
      },
    ],
  },
];

const router = new SimplrRouter({ routes });
router.init();
```

The pages will be imported on the first time they're loaded, saving you a lot of initial load time on your page.

**This also works with most bundlers, like Rollup**

To see this in action, you can open up the [Simplr Router docs](https://router.matsu.fi/) and look at the network tab in your dev tools.

![Code splitting example](/router-code-splitting.gif)

---

### Middleware

The last part of the Router I want to highlight in this blog is the extensibility.

With middleware support, the router can be easily modified without adding dependencies or extra code
into the main project.

Currently there are 2 official middlewares released:

- [A Preact Middleware](https://github.com/Simplr/simplr-router-preact-middleware)
  and
- [A React Middleware](https://github.com/Simplr/simplr-router-react-middleware)

These middlewares will add support for Preact and React projects respectively, and can be applied with just 2 lines of code:

```javascript
import SimplrRouter from "@simplr-wc/router";
import SimplrRouterPreactMiddleware from "@simplr-wc/router-preact-middleware";

const router = new SimplrRouter({ routes });
router.use(SimplrRouterPreactMiddleware());
router.init();
```

Adding these to the regular Simplr Router configuration will allow you to use the library in your
Preact projects too :)

A use case example can be found in the [Preact Middleware Repository](https://github.com/Simplr/simplr-router-preact-middleware/tree/main/example)
or in [this Codesandbox](https://codesandbox.io/s/sweet-dawn-schqt?file=/src/index.js)

More information about the Middlewares can be found in the [documentation](https://router.matsu.fi/recipes/middleware).

### Final word

Simplr Router is one of my most ambitious projects in open source Javascript and I really hope it
provides others with value as it has provided me already.

The aim of Simplr Router is to become one of the standard approaches in SPA routing, and to expands with
user input to provide more functionality and speed for developers and users alike, while remaining lightweight.

If you enjoyed this post, please make sure to check out Simplr Router in

- [NPM](https://www.npmjs.com/package/@simplr-wc/router)
- [GitHub](https://github.com/Simplr/simplr-router) (Maybe give it a ⭐ too ;) )
- [The Docs](https://router.matsu.fi/)
- Or [discuss it with me on Twitter](https://twitter.com/matsuuu_)
