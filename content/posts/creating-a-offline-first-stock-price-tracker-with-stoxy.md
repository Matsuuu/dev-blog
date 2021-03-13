---
title: "Creating a Offline First Stock Price Tracker With Stoxy"
date: 2021-03-13T14:01:19+02:00
---

At the start of this week, I released my state management library, Stoxy to the wild.

After gaining some tracktion from different outlets, I decided I should create a small demo project showcasing
the best parts about using Stoxy.

That's why I decided to create a small Offline-first application for tracking stock prices. It will be built
with Web Components and utilize the Stoxy library for handling the state, online and offline.

## Features

The app will be a installable PWA site, in which the user can enter different stocks
to follow the price of. The app will not have a separate backend, but only store the data in the
indexedDB via Stoxy.

After having entered stocks the user wants to follow, he can fetch the latest stock data with a
press of a button.

## Requirements

We're going to be creating a offline functional PWA web app, which means we're going to need the following:

- A manifest.json for creating the PWA
- A service worker for caching our pages and serving them offline
- An API key for our stock API

Let's get those two out of the way first before we get into development of the actual app.

## Initialization

I scaffolded my project using out own in-house scaffolding tool, `@simplr-wc/create` with the command `npm init @simplr-wc`.

This scaffolding tool is quite refined for our use cases and if you're following this post and developing yourself, I highly
recommend going with the [Open-WC](https://open-wc.org/) initializer

```bash
npm init @open-wc
```

Then selecting to scaffold a new application with linting and building tools pre-setup.

I named my project `stoxy-stock-project` so all the references to that name during this post will have to be changed
to match your naming scheme.

Now let's create the first file

#### Manifest.json

The manifest.json file is a required file for the web app to be recognized as a PWA.

The file specifies basic metadata about our PWA app for the device.

Following the MVP for the manifest.json file we end up with

```json
// manifest.json

{
  "name": "Stoxy Stock App Demo",
  "short_name": "Stoxy Stock",
  "lang": "en-US",
  "start_url": "/index.html",
  "display": "standalone",
  "theme_color": "#FFFFFF",
  "background_color": "#108ad4",
  "icons": [
    {
      "src": "icons/android-chrome-192x192.png",
      "sizes": "192x192"
    },
    {
      "src": "icons/android-chrome-512x512.png",
      "sizes": "512x512"
    }
  ]
}
```

This will do fine since we're only going to be using this for only personal use :)

#### Service Worker

Next up we will create a MVP of our service worker file to provide some offline functionality.

For large scale products or anything that is going into production for people to use, I would always use
[Workbox](https://developers.google.com/web/tools/workbox), since it makes everything a lot simpler,
but for this simple app, we're just going to be writing our implementation ourselves.

We will start by creating our service worker file, `worker.js`.

```javascript
// worker.js

const staticCacheName = "stockapp-cache-v1";
const filesToCache = [
  "/",
  "index.html",
  "src/stoxy-stock-project.js",
  "icons/android-chrome-192x192.png",
  "icons/android-chrome-512x512.png",
];

self.addEventListener("install", (e) => {
  console.log("[Service Worker] Install");
  e.waitUntil(
    caches.open(staticCacheName).then((cache) => {
      return cache.addAll(filesToCache);
    })
  );
});

self.addEventListener("activate", (e) => {
  console.log("[Service Worker] actived");
});

self.addEventListener("fetch", (event) => {
  event.respondWith(
    caches.match(event.request).then((cachedResponse) => {
      if (cachedResponse) {
        return cachedResponse;
      }
      return fetch(event.request);
    })
  );
});
```

First we declare the site cache name, then the files we want to cache in it.

In the install event (when the service worker is installed), we grab the event, and wait until the site cache has been
updated to contain all of the files we wanted to cache.

Next we set up a event listener for all of our `fetch`-events and make our service worker respond to all of
the requests matching our cached sites with the cached version. This is done so that when we have no internet
connectivity, we can still serve all of the files we need.

Next we will need to register our service worker.

In the initialization of our javascript, we create a function to register our service worker, if possible

```javascript
// src/service-worker-setup.js

export function register() {
  if ("serviceWorker" in navigator) {
    navigator.serviceWorker.register("/worker.js").then(() => {
      console.log("Service worker registered");
    });
  }
}
```

Now we're all set to start developing our offline-first stock app!

## Development

For development, we are using [LitElement](https://lit-element.polymer-project.org/) and [lit-html](https://lit-html.polymer-project.org/).
They make the development process of Web Components a breeze and work really nicely together with Stoxy.

So we are currently working with just 2 dependencies for the whole project: LitElement and Stoxy.

#### Initializing the base level app

First we are going to create the landing page for our app. It's going to contain all of the components required for
our app to run.

```javascript
// stoxy-stock-project.js

import { LitElement, html, css } from "lit-element";
import { getStockInfo } from "./stockApi";
import { persistKey, read } from "@stoxy/stoxy";
import "./stock-price-display";
import "./stock-adder";
import "@stoxy/repeat";
import { register } from "./service-worker-setup";

export default class StoxyStockProject extends LitElement {
  constructor() {
    super();
    persistKey("stockInfo", "stocks");
  }

  firstUpdated() {
    register();
  }

  async fetchStockInfo() {
    const stocks = await read("stocks");
    stocks.forEach((s) => getStockInfo(s));
  }

  render() {
    return html`
      <stock-adder></stock-adder>
      <button @click=${this.fetchStockInfo}>
        <svg
          xmlns="http://www.w3.org/2000/svg"
          width="24"
          height="24"
          viewBox="0 0 24 24"
        >
          <path
            d="M9 12l-4.463 4.969-4.537-4.969h3c0-4.97 4.03-9 9-9 2.395 0 4.565.942 6.179 2.468l-2.004 2.231c-1.081-1.05-2.553-1.699-4.175-1.699-3.309 0-6 2.691-6 6h3zm10.463-4.969l-4.463 4.969h3c0 3.309-2.691 6-6 6-1.623 0-3.094-.65-4.175-1.699l-2.004 2.231c1.613 1.526 3.784 2.468 6.179 2.468 4.97 0 9-4.03 9-9h3l-4.537-4.969z"
          />
        </svg>
      </button>
      <stoxy-repeat key="stocks" id="iStock" class="stock-area">
        <stock-price-display stock="iStock"></stock-price-display>
      </stoxy-repeat>
    `;
  }

  static get styles() {
    return css`
      :host {
        display: flex;
        flex-direction: column;
        font-size: 1.6rem;
        width: 100%;
        min-height: 100vh;
        background: #108ad4;
        align-items: center;
        justify-content: center;
        color: #fff;
        overflow: hidden;
      }

      .stock-area {
        width: 80%;
        justify-content: space-between;
        display: flex;
        flex-wrap: wrap;
      }

      button {
        border: none;
        background: none;
        margin-bottom: 5rem;
        fill: #fff;
        outline: none;
      }

      svg {
        width: 50px;
        height: 50px;
        cursor: pointer;
      }
    `;
  }
}

if (!customElements.get("stoxy-stock-project")) {
  customElements.define("stoxy-stock-project", StoxyStockProject);
}
```

In our constructor function, we initialize all of your Stoxy keys that we want to persist through page loads. Since we want
our stock data to be persisted for offline use, we want to persist the list of stocks we follow, and the stockinfo
about all of those stocks.

The firstUpdated method is called when the element is updated for the first time. At that point we can be fairly
sure that we can initialize our Service worker, and that's what we are going to do. We call the `register()` method we just
added to the `service-worker-setup.js` file.

After the initialization functions, we declare just one function: `fetchStockInfo`. This function will be used
for updating the stock data for the followed stocks. Inside the function we first read all of the stock names from
our stoxy cache, and after that, we iterate through each of them, fething the latest stock info of each.

Inside our render function we have our HTML which is going to build our site. We will go into further detail about the
components later in this post.

Here we saw our first look into using Stoxy. Inside the `fetchStockInfo` function we read the data from state with just
a simple promise call.

#### Adding a Stock to follow

Next up we are going to create the `stock-adder` component seen in the top level HTML.

We want this to be a simple form component, which takes the stock name from the form and adds it to the state.

```javascript
// src/stock-adder.js

import { LitElement, html, css } from "lit-element";
import { add } from "@stoxy/core";
import { getStockInfo } from "./stockApi";

class StockAdder extends LitElement {
  constructor() {
    super();
  }

  addStock(e) {
    e.preventDefault();
    const formData = new FormData(e.target);
    const stockName = formData.get("stock");
    add("stocks", stockName);
    this.shadowRoot.querySelector("input[type='text']").value = "";
    getStockInfo(stockName);
  }

  render() {
    return html`
      <form @submit=${this.addStock}>
        <h2>Add a stock to watch</h2>
        <input type="text" name="stock" placeholder="AMZN" />
        <input type="submit" value="Add stock" />
      </form>
    `;
  }

  static get styles() {
    return css`
      :host {
        margin-bottom: 2rem;
        width: 80%;
        display: flex;
        flex-direction: column;
        align-items: center;
        justify-content: center;
      }

      form {
        display: flex;
        flex-direction: column;
        align-items: center;
        justify-content: center;
      }

      input[type="text"] {
        width: 20rem;
        font-size: 2rem;
        text-align: center;
        border: 2px solid #fff;
        background: transparent;
        color: #fff;
      }

      input::placeholder {
        color: #fff;
        opacity: 0.4;
      }

      input[type="submit"] {
        margin-top: 1rem;
        border: 2px solid #fff;
        background: none;
        color: #fff;
        font-size: 1.6rem;
        cursor: pointer;
      }
    `;
  }
}

if (!customElements.get("stock-adder")) {
  customElements.define("stock-adder", StockAdder);
}
```

The component itself is quite a simple web component form: Some styling and a simple form with a single input field.

The part in which Stoxy comes in is when we handle the form submit.

Inside the `addStock` function which is called when the form submits, we first prevent the default submit of the
form, and after getting the stock name from the form data, we add it into the stoxy state with the
[add](https://stoxy.dev/docs/methods/add/) method.

Stoxy has many built in functions for handling state objects, and `add` is especially nice when you just want to
add to the already existing state array, without having the first read the data and then rewrite it.

After we have saved the stock name onto the state, we tell the stockApi class to fetch the stock data for the
newly added stock.

---

##### Note!

Instead of using a classic form, we could also use [Stoxy Form](https://stoxy.dev/docs/components/stoxy-form/).

In that case, we would have our state be represented by a array of JSON objects instead of just strings.

```javascript
// Using add & normal form
["GME", "TSLA"][
  // Using stoxy-form
  ({ stock: "GME" }, { stock: "TSLA" })
];
```

For simplicity's sake, we'll just use the add method here.

---

#### Fetching the stock data

Allright so now we have out basic view, and our add-stock-to-state -component. Next we will need to implement the API
to call the stock API for us and save the information onto the Stoxy state.

This is quite simple, since we get the exact examples from the API service.

For this project we will be using [RapidAPI](https://rapidapi.com/) to provide us with the API's we need.

Inside RapidAPI service we will be utilizing the [Yahoo Finance](https://rapidapi.com/apidojo/api/yahoo-finance1?endpoint=apiendpoint_ef6248fb-33e7-42ea-a8a6-58a7ef7f2177)
"get-summary" -api.

By changing the code snippet to fetch, we get the perfect example for our case:

```javascript
fetch(
  "https://apidojo-yahoo-finance-v1.p.rapidapi.com/stock/v2/get-summary?symbol=AMRN&region=US",
  {
    method: "GET",
    headers: {
      "x-rapidapi-key": "YOUR_RAPIDAPI_KEY",
      "x-rapidapi-host": "YOUR_RAPIDAPI_HOST",
    },
  }
)
  .then((response) => {
    console.log(response);
  })
  .catch((err) => {
    console.error(err);
  });
```

We can just copy and paste... I mean carefully type ourselves the code onto our stockApi.js file.
Let's also modify a few things while we're at it.

```javascript
// src/stockApi.js

import { update } from "@stoxy/core";

export async function getStockInfo(stockName) {
  const stockInfo = await fetch(
    `https://apidojo-yahoo-finance-v1.p.rapidapi.com/stock/v2/get-analysis?symbol=${stockName}&region=US`,
    {
      method: "GET",
      headers: {
        "x-rapidapi-key": "YOUR_RAPIDAPI_KEY",
        "x-rapidapi-host": "YOUR_RAPIDAPI_HOST",
      },
    }
  )
    .then((response) => response.json())
    .then((response) => {
      console.log(response);
      return response;
    })
    .catch((err) => {
      console.error(err);
    });

  update("stockInfo", (si) => {
    if (!si) si = {};
    si[stockName] = stockInfo;
    return si;
  });
}
```

First off we made the call into a function so that we can call it from anywhere in our codebase.

In the API call we also replace the hardcoded stock name with a variable gained as a parameter.

After we get our data from the API, we want to first pare it into JSON. After that we just console.log it to
make it easier to debug. You can of course omit the console log and the whole `then` part.

After we have gaines the stock info for the given stock, we call the [update](https://stoxy.dev/docs/methods/update/) function of Stoxy.
With the update function we can easily modify a part of a state object without explicitly reading the whole
state object and then explicitly writing it.

In the update delegate we check if we have `stockInfo` initialized in our state at all, and initialize it as needed.
After that we set the stock data from the API onto the key of the stock's name for easy fetching later on.

It was really easy to set the API data to be accessible across the whole application with just a quick `update` call to Stoxy.

#### Wrapping it all together

Quickly going through what we've built so far:

- Set up the application to function as a Offline ready PWA
- Created a form to submit a new Stock to follow
- Created main app component for our app
- Fetched the data from the API and saved it onto our persistent state

The last thing we now have to do is wrap it all together by creating the component to display
our stock price data.

#### Repeating data with Stoxy

You might have already spotted the [stoxy-repeat](https://stoxy.dev/docs/components/stoxy-repeat/) component
in our main app file. Inside the file we were using it to repeat out stock display component.

```html
<stoxy-repeat key="stocks" id="iStock" class="stock-area">
  <stock-price-display stock="iStock"></stock-price-display>
</stoxy-repeat>
```

Let's disect a little bit what's going on here:

With the stoxy-repeat -component we can easily repeat HTML structures according to our state data.

In the code snippet above, we are iterating through the "stocks" state object, and naming our iterator
objects "iStock" for "iteratedStock". 

With the component we can now reference the "iStock" word, and the component will replace it with the value
from the stock object itself. Since our stock state object is just a array of Strings, we don't need
any accessors like `iStock.name`  etc.

In the above example we set the `stock` attribute of our component to match the name of the current stock we're iterating through.

Next let's see how our stock display component handles this.

---

#### Stock display component

```javascript
// src/stock-price-display.js

import { LitElement, html, css } from 'lit-element';
import { remove, update } from '@stoxy/core';
import '@stoxy/object';

class StockPriceDisplay extends LitElement {
    static get properties() {
        return {
            stock: { type: String },

            higher: { type: Boolean, reflect: true },
            lower: { type: Boolean, reflect: true },
        };
    }

    getCacheKey() {
        return `st.${this.stock}.price.regularMarketPrice.fmt`;
    }

    removeStock() {
        remove('stocks', s => s === this.stock);
        update('stockInfo', si => {
            delete si[this.stock];
            return si;
        });
    }

    checkStockInfo(e) {
        if (!e.detail[this.stock]) return;
        const stockPriceData = e.detail[this.stock].price;
        if (stockPriceData.regularMarketPrice.raw > stockPriceData.regularMarketPreviousClose.raw) {
            this.higher = true;
            this.lower = false;
        } else {
            this.lower = true;
            this.higher = false;
        }
    }

    render() {
        return html`
            <stoxy-object key="stockInfo" prefix="st." @updated=${this.checkStockInfo}>
                <p>${this.stock}</p>
                <h2>${this.getCacheKey()}</h2>

                <button @click=${this.removeStock}>Unfollow stock</button>
            </stoxy-object>
        `;
    }

    static get styles() {
        return css`
            :host {
                border: 2px solid #fff;
                flex-basis: 40%;
                display: flex;
                flex-direction: column;
                align-items: center;
                justify-content: center;
                padding: 3rem 1rem;
                position: relative;
                background: rgba(255, 255, 255, 0.2);
                animation: fade-in 500ms;
                margin-bottom: 2rem;
            }

            @keyframes fade-in {
                from {
                    transform: translate(0, -50px);
                    opacity: 0;
                }
            }

            p,
            h2 {
                margin: 0.25rem;
            }

            :host([higher]) {
                color: springgreen;
            }

            :host([lower]) {
                color: #ff2929;
            }

            button {
                font-size: 1rem;
                position: absolute;
                bottom: 0.5rem;
                right: 0.5rem;
                border: none;
                background: none;
                color: red;
                font-weight: bold;
                cursor: pointer;
            }
        `;
    }
}

if (!customElements.get('stock-price-display')) {
    customElements.define('stock-price-display', StockPriceDisplay);
}

```

At first we declare our LitElement properties. We have our stock name there, but we also have 2 other attributes:
"higher" and "lower". We will use these to tell the component if the stock price is higher or lower than the
yesterday's close. The attributes have the property "reflect" set to true, which will make the component
reflect it's property onto the attribute of the HTML node too. These will ease us when styling our coponent.

After the properties we have our function which will construct us the cache key to access the stock
price data from our cached data. The price data of the stock is located in the property `[stockName].price.regularMarketPrice.fmt`.

Under our cache key function we have a function to remove said stock from being followed. Inside the function we have 
calls to the [update](https://stoxy.dev/docs/methods/update/) and [remove](https://stoxy.dev/docs/methods/remove/) -functions
of Stoxy. The remove-function removes the stock from the stocks array matching the name of the current stock.
The update method deleted the property from our stockInfo state object too, since we won't be needing that either.

The last function is for determining if the stock price is higher or lower than at yesterday's close.
The function call is triggered by the [stoxy-object](https://stoxy.dev/docs/components/stoxy-object/) -component
as it gets updated.

---

##### Rendering dynamic data

We are using stoxy-object inside the render function of the component to render the
data we want from the stock.

In our case, we are accessing the price data, and building the cache key in the `getCacheKey` -method.

We could add a infinite amount of accessors onto the state data inside our stoxy-object component and
they would get replaced with the corresponding values, similiar to the stoxy-repeat component.


#### Wrapping it all up

We now have our stock price tracking PWA ready to be used, and with it we could check the prices 
even when we're offline. We have a manual update button on our front page to update the data when
we want to update it, instead of fetching it every time we open the app.

Let's take a quick recap of how we achieved all of this and how Stoxy made it extremely simple:

- Stoxy provided us with a global state for both the stocks to follow and the API data
  - This allowed us to fetch the data from the API once, and then just re-use the same data, instead of capping
  our quota from just the development process.
  - This allows us to access the data we have already fetched, even if we go offline
  - It allows us to focus on the development instead of focusing on the data flow of a one-directional system

- Stoxy didn't create any extra over head
  - With Stoxy we didn't need to create any centralized service to handle all of our state
  - Neither did we need to setup any kind of environments or classes to get up and running
  - Stoxy worked perfectly from the get-go with the plug and play system it was built to support.

- Stoxy allowed us to utilize th IndexedDB without the headache of using it directly
  - A lot of developers might stray away from using API's like the IndexedDB due to the workflow it
  usually introduces to the app. With Stoxy the whole process is opt-in and abstracted behind the same
  functions you use for general state management
  - By opting in to persist the data with the `persistKeys("stocks", "stockInfo")`  we were all set
  for using the application's data offline on top of having the state react to all of the updates.

- Stoxy API didn't force a steep learning curve
  - With a lot of libraries and posts being created about easing the use of pre-existing state management systems,
  there should be no need for any large tutorials on using Stoxy: The API is built with simple 1-2 parameter functions
  which do exactly what you think they would do.
  - The [docs](https://stoxy.dev/docs/) provide examples of every normal case of use, but doesn't limit the use of 
  how far you can take Stoxy

### Final Word

I hope you enjoyed this quick look at how to create quick, reactive and functional Offline apps with Stoxy, and
if you would like to try Stoxy today, you can easily get started by looking up the guides at [Stoxy.dev](https://stoxy.dev/).

If you end up enjoying using Stoxy, give me a shout at [Twitter](https://twitter.com/matsuuu_), or [GitHub](https://github.com/Matsuuu/stoxy)
and if you really enjoy the library and would like to see it grow, give it a Star on the [GitHub](https://github.com/Matsuuu/stoxy) page!


Stoxy grew fast in popularity in the first week of it's release and I hope to keep up the growth with new features and
even more powerful workflows. I will also be using Stoxy for all of my future projects which need a state management system
and I'd be humbled if you would give it a go too :)

