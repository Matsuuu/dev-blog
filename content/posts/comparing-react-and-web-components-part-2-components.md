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

To run our dev environment, we need two terminals: One navigating to the component directory and serving the
component. The other one navigating to the example directory and serving that project instead. This made
me wonder if they could've somehow combined these two tasks with something like [Concurrently](https://www.npmjs.com/package/concurrently).
Nevertheless the dev experience from here onwards was pretty painless.

We are provided with a nice example configuration for the component, and we are able to easily get to work. Our final
component source looked like this.

```js
import React, { useState } from 'react';
import styles from './index.module.css';

const placeKittenUrl = 'http://placekitten.com/{width}/{height}';

export default function CatImageViewer() {
    const [imageWidth, setImageWidth] = useState(0);
    const [imageHeight, setImageHeight] = useState(0);
    const [currentImage, setCurrentImage] = useState(null);

    const getNewCatImage = () => {
        const searchUrl = placeKittenUrl.replace('{width}', imageWidth).replace('{height}', imageHeight);
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
