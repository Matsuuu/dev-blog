---
title: 'Comparing React and Web Components. Part 2: Components'
date: 2020-08-19T15:00:00+03:00
---

This is part 2 of my series comparing Web Components and React. You can find the first part [here](https://matsu.fi/posts/comparing-react-and-web-components-part-1-starters/)

Before we get started, I'll clear out some possible misconceptions from the last post:

-   OpenWC is not LitElement. LitElement is a Open source library written by the Polymer team and the open source contributors while
    OpenWC is a community aimed at providing recommendations for web component development.
-   LitElement is a base class that makes use of the lit-html library.
-   lit-html is a templating library used for creating fast html templates.

Allright. Now let's continue where we left off.

---

### Comparing Component development

In today's post we'll be comparing building Javascript components in 3 different evironments:

-   React
-   Lit-Element
-   HTMLElement (vanilla)

We will go through a simple implementation of a small form, in which the user can insert the desired width and height
of a random cat picture they desire and then receive it by our code querying the [PlaceKitten](https://placekitten.com/) API

Our finished application will be look something along the lines of the picture below.

![Cat Image Viewer](/cat-viewer-example.png)

It's beautiful, right? I didn't spend much time on css, since only the usage of css is relevant to this post,
not the stylized output of it.

Let's first dive into how we went through this task in React

---

### The React Implementation

#### Creation

For the React implementation, I found a lot of differing material around online on what was the
best way to create React components / libraries. I ended up settling on [create-react-library](https://github.com/transitive-bullshit/create-react-library)
due to it having a nice >3k stars on Github and a straight forward starter kit.

We first start by running the instructed command `npx create-react-library`, which is nicely familiar
to the `create-react-app` a lot of people use, including us in the last post.

Running this command creates us a project, or actually 2 seperate projects.

We got ourselves a project for our component development, and also a project for demoing our
component in a "real" environment.

This works by the first project building the component, and serving it with the other needed libraries
by symlinking them to the node_modules of the example-directory.

This allows our demo project to import it like a normal package would

```javascript
import CatImageViewer from 'cat-image-react-viewer';
```

The node_modules of the demo project are symlinked from the top level project, which is the component project.

```bash
> ll example/node_modules/

total 20
drwxrwxr-x 5 matsu matsu 4096 Aug 13 13:56  ./
drwxrwxr-x 5 matsu matsu 4096 Aug 13 13:56  ../
drwxrwxr-x 4 matsu matsu 4096 Aug 13 13:56 '@babel'/
drwxrwxr-x 2 matsu matsu 4096 Aug 13 13:56  .bin/
drwxrwxr-x 4 matsu matsu 4096 Aug 13 13:56  .cache/
lrwxrwxrwx 1 matsu matsu    5 Aug 13 13:56  cat-image-react-viewer -> ../../
lrwxrwxrwx 1 matsu matsu   24 Aug 13 13:56  react -> ../../node_modules/react/
lrwxrwxrwx 1 matsu matsu   28 Aug 13 13:56  react-dom -> ../../node_modules/react-dom/
lrwxrwxrwx 1 matsu matsu   32 Aug 13 13:56  react-scripts -> ../../node_modules/react-scripts/
```

#### Development

To run our dev environment, we need two terminals: One navigating to the component directory and serving the
component. The other one navigating to the example directory and serving that project instead. I know a lot of developers
who solely use the VSCode terminal, making this quite a hassle.

This made me wonder if they could've somehow combined these two tasks with something like [Concurrently](https://www.npmjs.com/package/concurrently).
Nevertheless the dev experience from here onwards was pretty painless.

We are provided with a nice example configuration for the component, and we are able to easily get to work. Our final
component source looked like this.

```javascript
import React, { useState } from 'react';
import styles from './index.module.css';

const placeKittenUrl = 'http://placekitten.com/{width}/{height}';

export default function CatImageViewer() {
    const [imageWidth, setImageWidth] = useState(0);
    const [imageHeight, setImageHeight] = useState(0);
    const [currentImage, setCurrentImage] = useState(null);

    const getNewCatImage = () => {
        const searchUrl = placeKittenUrl
            .replace('{width}', imageWidth)
            .replace('{height}', imageHeight);
        setCurrentImage(searchUrl);
    };

    return (
        <div className={styles.formWrapper}>
            <div>
                <p>Enter the dimensions of the desired cat image</p>
                <input
                    type="number"
                    placeholder="Width"
                    id="image-width"
                    onInput={(e) => setImageWidth(e.target.value)}
                />
                <input
                    type="number"
                    placeholder="Height"
                    id="image-height"
                    onInput={(e) => setImageHeight(e.target.value)}
                />
                <button type="button" onClick={getNewCatImage}>
                    Search
                </button>
            </div>
            {currentImage && <img alt="" src={currentImage} />}
        </div>
    );
}
```

with the css module looking like this:

```css
.formWrapper div {
    display: flex;
    flex-direction: column;
}

.formWrapper input,
.formWrapper button {
    font-size: 2rem;
    margin: 0 0 1rem;
    width: 10rem;
}
```

I created the component using React Hooks since they seem to be the hottest thing right now in the React world.
This allowed us to fairly concisely create the component.

I'm by no means a React master, but this is perfect for emulating the developer experience
of a new dev wanting to create their first component.

The starter had a `index.module.css` set up from the start, and from what I've looked at
React projects, seperate css files for components are fairly usual (compared to the Web Component way of
having the css in js).

As you might remember from the last post, the `create-react-app` had a huge footprint in the
package size on a vanilla installation. I was surprised and happy as I noticed that after building
we weren't greeted by a 500KB package.

```bash
> npm run build

> du -sh dist/
24K     dist/
```

I'm guessing this is partly due to the fact that our component project doesn't build any of it's dependencies
with it. In the package.json we have no libraries listed as dependencies. However we have React
listed as a peerDependency meaning that our package is forever doomed to be run only by
other React projects.

A quick look inside the dist folder also explains why the size of the package is 24K: the project
builds itself as a commonjs and as and es package.

```bash
> ll dist/

total 28
drwxrwxr-x 2 matsu matsu 4096 Aug 13 13:55 ./
drwxrwxr-x 7 matsu matsu 4096 Aug 13 13:56 ../
-rw-rw-r-- 1 matsu matsu  149 Aug 19 17:12 index.css
-rw-rw-r-- 1 matsu matsu 1833 Aug 19 17:12 index.js
-rw-rw-r-- 1 matsu matsu 2595 Aug 19 17:12 index.js.map
-rw-rw-r-- 1 matsu matsu 1607 Aug 19 17:12 index.modern.js
-rw-rw-r-- 1 matsu matsu 2584 Aug 19 17:12 index.modern.js.map
```

This is fairly convenient, but also doubles the package size, in a case that both of these versions
get published.

If we take a look at our example folder, we can see how the project would be imported in a React environment:

```js
import React from 'react';

import CatImageViewer from 'cat-image-react-viewer';
import 'cat-image-react-viewer/dist/index.css';

const App = () => {
    return <CatImageViewer />;
};

export default App;
```

We notice that to make use of the styles of our component, we are forced to import the css package
as it's own import. This means that to build this project, we are required to have a css bundler
in our project.

However React usually is built with something like Webpack or Rollup, so this shouldn't
cause too much overhead. In a buildless environment however this would cause a headache or two.

Overall after finding and setting up the environment, the development was fairly straight forward. The starter
provided us with commands for publishing the package. The one downside here is like I said, that the package can now
only be used if React is used in the project.

#### Output

Last thing we will take a look at is the output of the build process.

As said earlier, we have 5 files, from which

-   1 is a css file
-   2 are js files
-   2 are js map files

Let's get the easiest out of the way and look inside the css file:

```css
._index-module__formWrapper__r8sB5 div {
    display: flex;
    flex-direction: column;
}

._index-module__formWrapper__r8sB5 input,
._index-module__formWrapper__r8sB5 button {
    font-size: 2rem;
    margin: 0 0 1rem;
    width: 10rem;
}
```

We can see that the styles have stayed untouched, except for the classname. As we never named a css selector outside of the
call `styles.formWrapper`, we can see that the build process generated a unique identifier for the class.

I see two cases here which I would like to highlight:

-   Creating convoluted class names makes it harder for test automation to create readable tests that don't break after a new build
-   `index-module__formWrapper__r8sB5` isn't really as descriptive to read if looking at the source from the browser as `form-wrapper` would be.

Next we'll check out the `index.js` file:

```javascript
function _interopDefault(ex) {
    return ex && typeof ex === 'object' && 'default' in ex ? ex['default'] : ex;
}

var React = require('react');
var React__default = _interopDefault(React);

var styles = { formWrapper: '_index-module__formWrapper__r8sB5' };

var placeKittenUrl = 'http://placekitten.com/{width}/{height}';
function CatImageViewer() {
    var _useState = React.useState(0),
        imageWidth = _useState[0],
        setImageWidth = _useState[1];

    var _useState2 = React.useState(0),
        imageHeight = _useState2[0],
        setImageHeight = _useState2[1];

    var _useState3 = React.useState(null),
        currentImage = _useState3[0],
        setCurrentImage = _useState3[1];

    var getNewCatImage = function getNewCatImage() {
        var searchUrl = placeKittenUrl
            .replace('{width}', imageWidth)
            .replace('{height}', imageHeight);
        setCurrentImage(searchUrl);
    };

    return /*#__PURE__*/ React__default.createElement(
        'div',
        {
            className: styles.formWrapper,
        },
        /*#__PURE__*/ React__default.createElement(
            'div',
            null,
            /*#__PURE__*/ React__default.createElement(
                'p',
                null,
                'Enter the dimensions of the desired cat image'
            ),
            /*#__PURE__*/ React__default.createElement('input', {
                type: 'number',
                placeholder: 'Width',
                id: 'image-width',
                onInput: function onInput(e) {
                    return setImageWidth(e.target.value);
                },
            }),
            /*#__PURE__*/ React__default.createElement('input', {
                type: 'number',
                placeholder: 'Height',
                id: 'image-height',
                onInput: function onInput(e) {
                    return setImageHeight(e.target.value);
                },
            }),
            /*#__PURE__*/ React__default.createElement(
                'button',
                {
                    type: 'button',
                    onClick: getNewCatImage,
                },
                'Search'
            )
        ),
        currentImage &&
            /*#__PURE__*/ React__default.createElement('img', {
                alt: '',
                src: currentImage,
            })
    );
}

module.exports = CatImageViewer;
//# sourceMappingURL=index.js.map
```

Allright so after we get over the fact that this code is quite unreadable, especially the render part, we can
take a look at a few parts.

-   Why are all of our const's turned into `var`s? Out of all the possibilities, a var.
    -   const is supported by all browsers back to IE11 (which by the way is not even supported soon anymore)
-   If a developer was to inspect this element straight from the source, without knowing how React works, it would be quite hard to know
    what's going on with the hooks.
    -   In the browser this is resolved by providing a source map of the file, provided your browser supports it and has it enabled.
    -   Source maps aren't foolproof and might introduce some problems while debugging

Next let's look inside the other generated javascript file: `index.modern.js`

```javascript
import React, { useState } from 'react';

var styles = { formWrapper: '_index-module__formWrapper__r8sB5' };

const placeKittenUrl = 'http://placekitten.com/{width}/{height}';
function CatImageViewer() {
    const [imageWidth, setImageWidth] = useState(0);
    const [imageHeight, setImageHeight] = useState(0);
    const [currentImage, setCurrentImage] = useState(null);

    const getNewCatImage = () => {
        const searchUrl = placeKittenUrl
            .replace('{width}', imageWidth)
            .replace('{height}', imageHeight);
        setCurrentImage(searchUrl);
    };

    return /*#__PURE__*/ React.createElement(
        'div',
        {
            className: styles.formWrapper,
        },
        /*#__PURE__*/ React.createElement(
            'div',
            null,
            /*#__PURE__*/ React.createElement(
                'p',
                null,
                'Enter the dimensions of the desired cat image'
            ),
            /*#__PURE__*/ React.createElement('input', {
                type: 'number',
                placeholder: 'Width',
                id: 'image-width',
                onInput: (e) => setImageWidth(e.target.value),
            }),
            /*#__PURE__*/ React.createElement('input', {
                type: 'number',
                placeholder: 'Height',
                id: 'image-height',
                onInput: (e) => setImageHeight(e.target.value),
            }),
            /*#__PURE__*/ React.createElement(
                'button',
                {
                    type: 'button',
                    onClick: getNewCatImage,
                },
                'Search'
            )
        ),
        currentImage &&
            /*#__PURE__*/ React.createElement('img', {
                alt: '',
                src: currentImage,
            })
    );
}

export default CatImageViewer;
//# sourceMappingURL=index.modern.js.map
```

Now this one is looking more like what we wrote, and that's mostly because this is the es module version of the
built code. We see that our `const`s have been untampered with this time.

Our project was fairly simple, so the source code can't be analyzed deeper, but I'd imagine that in more complex systems,
you could find more to talk about these built files.

Next we'll look at a little more vanilla approach with Lit Element

---

### The Lit Element implementation

#### Creation

Moving on to [Lit Element](https://lit-element.polymer-project.org/), we can make use of the same [Open WC](https://open-wc.org/) init
command we used when creating a sample project. The initializer will prompt us to create either an application or a component.
In our case we want to obviously choose the Web Component -selection.

```bash
npm init @open-wc
```

After running the initializer, and selecting our configuration, we have a nice sample project like we had in the React component
example. The difference here is that we can get to developing with just running a single `npm start` instead of running our seperate demo
instance. The twist is that we have to relatively call our component instead it being in our node_modules.

#### Development

Running `npm start` starts the [es-dev-server](https://www.npmjs.com/package/es-dev-server) and serves a demo `index.html` file
appending our newly initialized Web Component. In the case we would create a Typescript component, the command would run es-dev-server
and tsc in parallel using concurrently. This is really nice allowing users to run the whole environment with just a single command.

After running the project and cleaning up the example files, we ended up writing our LitElement implementation as follows:

```javascript
import { html, css, LitElement } from 'lit-element';

const placeKittenUrl = 'http://placekitten.com/{width}/{height}';

export class CatImageLitViewer extends LitElement {
    static get properties() {
        return {
            imageWidth: { type: Number },
            imageHeight: { type: Number },
            currentImage: { type: String },
        };
    }

    static styles = css`
        .form-wrapper {
            display: flex;
            flex-direction: column;
        }
        button,
        input {
            font-size: 2rem;
            margin: 0 0 1rem;
            width: 10rem;
        }
    `;

    constructor() {
        super();
        this.imageWidth = 0;
        this.imageHeight = 0;
        this.currentImage = null;
    }

    getNewCatImage() {
        const searchUrl = placeKittenUrl
            .replace('{width}', this.imageWidth)
            .replace('{height}', this.imageHeight);
        this.currentImage = searchUrl;
    }

    render() {
        return html`
            <div class="form-wrapper">
                <p>Enter the dimensions of the desired cat image</p>
                <input
                    type="number"
                    placeholder="Width"
                    id="image-width"
                    @input=${(e) => {
                        this.imageWidth = e.target.value;
                    }}
                />
                <input
                    type="number"
                    placeholder="Height"
                    id="image-height"
                    @input=${(e) => {
                        this.imageHeight = e.target.value;
                    }}
                />
                <button type="button" @click=${this.getNewCatImage}>
                    Search
                </button>
            </div>
            ${this.currentImage && html`<img alt="" src="${this.currentImage}" />`}
        `;
    }
}

customElements.define('cat-image-lit-viewer', CatImageLitViewer);
```

Compared to our React example, the LitElement version is quite similiar. The differences are mainly in state management.

In React we had hooks handling our states. With LitElement we save our state objects as properties on our class, and then
initialize them inside the constructor. It's a few lines more of code, but it also gives us a nicely typed API for our component,
even in Javascript.

The next difference is that we have our styles _inside_ of the js file, unlike in React where we used a css module file.
Our style selectors are also a bit more relaxed, since we are developing our component into shadow DOM, allowing for an encapsulated
handling of styles.

Also differentiating us from the react project is the resulting DOM tree. In the React project we were forced to have a `div` -element
wrapping everything in our component. In the Web Component version, we will have a named Web Component wrapping our element, making
the need for a wrapping div obselete.

```html
<!-- React -->
<div class="_index-module__formWrapper__r8sB5">
    <div>
        <p>Enter the dimensions of the desired cat image</p>
        <input type="number" placeholder="Width" id="image-width" />
        <input type="number" placeholder="Height" id="image-height" />
        <button type="button">
            Search
        </button>
    </div>
</div>

<!-- LitElement -->
<cat-image-lit-viewer>
    <div class="form-wrapper">
        <p>Enter the dimensions of the desired cat image</p>
        <input type="number" placeholder="Width" id="image-width" />
        <input type="number" placeholder="Height" id="image-height" />
        <button type="button">
            Search
        </button>
    </div>
</cat-image-lit-viewer>
```

That's mostly the difference in the code side. Both of our implementations have a nicely structured render with some conditional
rendering thrown in there, and our components handle events with event listener properties (`onInput` and `onClick` with React, `@input`, `@click` on LitElement).

Now that we have our component, we should be ready to build.

#### Output

... Except that we won't do that.

---

Reading the open-wc recommendations on building, in the section about creating components, it says:

> If you are building a reusable component or library we recommend publishing code that runs without modifications on the latest browsers. You should not bundle in any dependencies or polyfills, or build to a very old version of JavaScript such as ES5. This way, consuming projects can decide which polyfills to load and which JavaScript version to target based on browser support.

> In practical terms, this means publishing standard ES modules and standard JavaScript features implemented in modern browsers, like Chrome, Safari, Firefox, and Edge. We recommend buildless development, so unless you are using very cutting edge features, you can actually just publish your source code as is.

Meaning in short that we can ship our Web Component _as is_. Isn't that cool?

Also since we're not building our package, we don't need to ship any source maps.

If we were to ship our Web Component, we would only need to ship the one js file, and tag LitElement as the dependency.
LitElement being a fairly small library (318kB unpacked), our total project size would be in total around 320kB.

```bash
> du -sh src/CatImageLitViewer.js

4.0K    CatImageLitViewer.js
```

This is, if we were not already using LitElement in our project. If we are already writing a LitElement project, our new package would
only bring the 4KB with it to the project. And this is unminified.

_For comparison_: If we wanted to add a single React Component to a non-react project, the minimal setup would require both React
and React-DOM, which together weight in at 204kB + 3MB unpackaged.

Also a plus in the LitElement approach is that we won't have to build the project in the implementing side either, since
the css in baked into the js file, therefore eliminating the need for a css loader (unlike in the React version).

Overall when creating Web Components with LitElement, we ended up with 6x smaller package size (4kb vs 24kb total), and
about 9x smaller total size with dependencies.

Next we'll take a look at fully buildless and dependencyless implementation using the browser's own HTMLElement.

---

### HTMLElement implementation

#### Creation

As we're building a project as vanilla as possible, we don't really need a initializer for this implementation.

We can just start by creating a folder, running npm init and creating the necessary files

```bash
mkdir cat-image-vanilla-viewer
cd cat-image-vanilla-viewer
npm init
touch index.html index.js
```

Now we can populate our index.html with the minimal setup needed for development:

```html
<html>
    <head> </head>
    <body>
        <cat-image-vanilla-viewer></cat-image-vanilla-viewer>
        <script src="./index.js" type="module"></script>
    </body>
</html>
```

Real simple and smooth. Just like Vanilla ice cream.

We could serve our project with any http server implementation, from [http-server](https://www.npmjs.com/package/http-server) to
[es-dev-server](https://www.npmjs.com/package/es-dev-server), since it's just a simple html + js file setup. For this post
I just added `es-dev-server` as a dev dependency, as it won't affect the package size that way.

Our package.json is now also in it's minimal setup:

```json
{
    "name": "cat-image-vanilla-viewer",
    "version": "1.0.0",
    "description": "",
    "main": "index.js",
    "scripts": {
        "start": "es-dev-server --app-index index.html --node-resolve --watch"
    },
    "author": "",
    "license": "ISC",
    "devDependencies": {
        "es-dev-server": "^1.57.2"
    }
}
```

Now we can just run `npm start` and get a nice, hot-reloading developer experience.

#### Development

As we use es-dev-server, the development setup and workflow are quite similiar to the Lit-Element example.

Now that we have the environment setup, we can write our component. The implementation is just a tad bit longer than the
ones we made using the libraries.

```javascript
const template = document.createElement('template');
template.innerHTML = `
<style>
    .form-wrapper {
        display: flex;
        flex-direction: column;
    }
    button,
    input {
        font-size: 2rem;
        margin: 0 0 1rem;
        width: 10rem;
    }
</style>
<div class="form-wrapper">
    <p>Enter the dimensions of the desired cat image</p>
    <input
        type="number"
        placeholder="Width"
        id="image-width"
    />
    <input
        type="number"
        placeholder="Height"
        id="image-height"
    />
    <button type="button" >Search</button>
</div>
`;

const placeKittenUrl = 'http://placekitten.com/{width}/{height}';

export default class CatImageVanillaViewer extends HTMLElement {
    constructor() {
        super();
        this.imageWidth = 0;
        this.imageHeight = 0;
        this.currentImage = null;

        const root = this.attachShadow({ mode: 'open' });
        root.appendChild(template.content.cloneNode(true));
    }

    connectedCallback() {
        this.shadowRoot
            .querySelector('#image-width')
            .addEventListener('input', (e) => (this.imageWidth = e.target.value));

        this.shadowRoot
            .querySelector('#image-height')
            .addEventListener('input', (e) => (this.imageHeight = e.target.value));

        this.shadowRoot
            .querySelector('button')
            .addEventListener('click', this.getNewCatImage.bind(this));
    }

    getNewCatImage() {
        const searchUrl = placeKittenUrl
            .replace('{width}', this.imageWidth)
            .replace('{height}', this.imageHeight);
        this.currentImage = searchUrl;

        let imageElem = this.shadowRoot.querySelector('img');
        if (!imageElem) {
            imageElem = document.createElement('img');
            this.shadowRoot.appendChild(imageElem);
        }
        imageElem.src = searchUrl;
    }
}

customElements.define('cat-image-vanilla-viewer', CatImageVanillaViewer);
```

Let's start going through what's different in this implementation. As you might see from the start, quite a lot has changed.

Firstly we're working with [templates](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/template).
Templates are used by `lit-html` under the hood too, but here we are creating them manually and
populating the contents here instead of inside the `render` function, which our component doesn't have.

The templates are "a mechanism for holding HTML that is not to be rendered immediately when a page is loaded, but
may be instantiated subsequently during runtime using Javascript" (From the template MDN page).

> Think of a template as a content fragment that is being stored for subsequent use in the document.
> While the parser does process the contents of the \<template\> element while loading the page, it does so only
> to ensure that those contents are valid; the element's contents are not rendered, however.

The Template elements enable us to write markup templates that are not rendered immeadiately and can then be
reused multiple times as the basis of a custom element's structure.

In our template element we declare our styling, and also the HTML structure of our component. The content
is basically what we would normally put inside the `render` -function.

Next up we create the class itself. We extend the [HTMLElement](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement),
which is a part of the Javascript Web API.
Any class extending HTMLElement represents a HTML element. Some Web Components might implement this interface directly,
while others might implement another interface, which inherits this interface.

Our Web Component has it's own built-in lifecycle callbacks of which we use some in this example.
The full list of the callbacks is as follows:

-   **_connectedCallback_**: Invoked when the custom element is first connected to the document's DOM.
-   **_disconnectedCallback_**: Invoked when the custom element is disconnected from the document's DOM.
-   **_adoptedCallback_**: Invoked when the custom element is moved to a new document.
-   **_attributeChangedCallback_**: Invoked when one of the custom element's attributes is added, removed, or changed

_Source: [MDN](https://developer.mozilla.org/en-US/docs/Web/Web_Components)_

Inside our constructor we initialize our properties with their default values. We also attach the shadow DOM
to our element to create encapsulation for our Web Component.
After creating the shadowRoot, we append a copy of our template inside it.

After `connectedCallback` is called, we can be sure that our element has hit the DOM, making it safe to attach events
to our element. So that's what we do inside our connectedCallback.

We could handle the onclick and oninput events inline, but while creating vanilla components, I prefer attaching the elements
manually inside the javascript instead of appending them inline into the html.

After that we are only left with the appending of the cat image. Different from the other cases, here we are not able to
just easily conditionally render our element, well at least not as easily as in the other libraries.

In our instance, it's easier to just append the image element into the element's DOM tree as the image is
added for the first time. After that we just `querySelector` our image element, and manipulate it's
src element manually.

#### Output

Once again, since this is fully vanilla, our component is supported by browsers without need for building.

Let's look at the size of our package, which we are going to be shipping for comparison with the others:

```bash
> du -sh index.js

4.0K    index.js
```

Well look at that. It's the same size as our LitElement implementation, but without adding the LitElement and lit-html
libraries to your project (in case you're not already using them).

And as we had in LitElement, in vanilla we don't need to provide a source maps, since we'll be shipping a unbuilt,
unminified version of our package.

Building the package in full vanilla also allows us to easily import this to literally any page we want, by just
importing the package from a JS CDN like [unpkg](https://unpkg.com/) and just adding the HTML element to the page.
No external tooling required.

Of course this comes with some drawbacks, which don't become relevant from this kind of a simple package.

With Vanilla Web Components

-   You can't pass other than string-based data via HTML
    -   But you can do it by passing the properties in javascript

```javascript
// Example
const profileObject = { id: 1, name: 'Matsu' };
document.querySelector('my-element').profile = profileObject;
```

-   You need to register attribute change callback yourself with callbacks like `attributechangedcallback`
-   You can get confused between attributes and properties
    -   But this is solved by just studying the subject, and in the end isn't all that complicated
-   You might need to write some custom update logic

But for creating simple Web Components like the one we created, some of these problems might not even surface.

---

### Verdict

So to recap, you should write a X component when...

-   Write a React component if you don't have any other choice
-   Write a LitElement component if you are going to be building anything more complicated, and can
    afford to force your component's developers to use slight tooling like Rollup.
    -   ... at least until bare module specifiers are supported globally.
-   Write a Vanilla component if you're building a simple component, that doesn't require some of the more complicated
    functionalities of the libraries and want your component to be able to be run absolutely (well almost) everywhere.

So the next time you are building a Javascript component, take a second to evaluate on which scale you want to
build it, and what level of dependencies you want to tie your users to.

I hope you all are enjoying the series. Please provide me feedback about the posts and correct me if I said something
wrong in writter at [@matsuuu\_](https://twitter.com/matsuuu_) and if you're enjoying the series, please
send me a DM and let me know :)

Until next time, when we'll be discussing more Web Components.
