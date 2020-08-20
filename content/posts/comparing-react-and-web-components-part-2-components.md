---
title: 'Comparing React and Web Components. Part 2: Components'
date: 2020-08-19T15:00:00+03:00
---

This is part 2 of my series comparing Web Components and React. You can find the first part [here](https://matsu.fi/posts/comparing-react-and-web-components-part-1-starters/)

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

... Except that we won't do that.

---

Reading the open-wc recommendations on building, in the section about creating components, it says:

> If you are building a reusable component or library we recommend publishing code that runs without modifications on the latest browsers. You should not bundle in any dependencies or polyfills, or build to a very old version of JavaScript such as ES5. This way, consuming projects can decide which polyfills to load and which JavaScript version to target based on browser support.

> In practical terms, this means publishing standard ES modules and standard JavaScript features implemented in modern browsers, like Chrome, Safari, Firefox, and Edge. We recommend buildless development, so unless you are using very cutting edge features, you can actually just publish your source code as is.

Meaning in short that we can ship our Web Component _as is_. Isn't that cool?

If we were to ship our Web Component, we would only need to ship the one js file, and tag LitElement as the dependency.
LitElement being a fairly small library (318kB unpacked), our total project size would be in total around 320kB.

```bash
> du -sh src/CatImageLitViewer.js

4.0K    CatImageLitViewer.js
```

This is, if we were not already using LitElement in our project. If we are already writing a LitElement project, our new package would
only bring the 4KB with it to the project.

_For comparison_: If we wanted to add a single React Component to a non-react project, the minimal setup would require both React
and React-DOM, which together weight in at 204kB + 3MB unpackaged.

Also a plus in the LitElement approach is that we won't have to build the project in the implementing side either, since
the css in baked into the js file, therefore eliminating the need for a css loader (unlike in the React version).

Overall when creating Web Components with LitElement, we ended up with 6x smaller package size (4kb vs 24kb total), and
about 9x smaller total size with dependencies.

Next we'll take a look at fully buildless and dependencyless implementation using the browser's own HTMLElement.

---

### HTMLElement implementation
