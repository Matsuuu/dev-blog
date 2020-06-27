---
title: 'Porting Libraries to Web Components'
date: 2020-06-27T11:00:55+03:00
---

As Web Components become a part of the [Web Standard](https://html.spec.whatwg.org/multipage/custom-elements.html), more libraries are being
created with Web Components instead of just exposing the API of said library.

But what about the libraries created before the rise of
Web Components? Can they be ported into Web Components, and what kind of a workload would this be? This is what we'll be discussing today.

### Preface

So why would we want to create a Web Component equivalent of a library if there already is an existing implementation?
Well the first thing that comes to mind is the ease-of-use and ease-of-implementation. By creating a Web Component of a library the
implementation of said functionality changes from

```html
<p id="paragraph-to-highlight">Please highlight me with the library</p>

<script src="/library/api.js"></script>
<script>
    window.onload = () => {
        const par = document.querySelector('#paragraph-to-highlight');
        HighlightLibrary.highlight(par);
    };
</script>
```

To a more simplistic

```html
<highlight-library>
    <p>Please highlight me with the library</p>
</highlight-library>

<script src="/library/wc.js" type="module"></script>
```

There are multiple pro's to this:

-   Less code
-   No pollution of the ID/class space
-   No arbitary code snippets tied to window load events etc.

And many more. Of course some of these problems might be eliminated by using modern frameworks or libraries, in which case
the javascript is located in seperate files and not in the HTML, but still in that case you would need to write something like
`initializeHighlights()` and call it on page load either way.

By automatic this process inside the Web Component, and making the component responsible of it's work, we can ease the development
process by a lot.

### Getting started

In this post I'll use my port of a library called [Rough Notation](https://github.com/pshihn/rough-notation) as an example.

Rough Notation is a small library used for highlighting content on a html page with nice hand drawn animations.

The regular use of Rough Notation looks like this:

```js
import { annotate } from 'rough-notation';

const element = document.querySelector('#myElement');
const annotation = annotate(element, { type: 'underline' });
annoation.show();
```

To get the same functionality with [Vanilla Rough Notation](https://github.com/Matsuuu/vanilla-rough-notation), the Web Component port I wrote,
you would only need to write:

```html
<rough-notation showOnLoad type="underline">
    <p>Please underline me</p>
</rough-notation>
```

This creates a nice environment for developing using the library, since no extra javascript is necessary. You can just wrap the elements in a
Web Component, and the functionality is applied out of the box.

This makes for a faster workflow since the developer will not have to write initializing functions in their codebase, and can just apply the effect on the
element as the element is created.

This way of working is also more beginner-friendly, since it required 0 javascript knowledge to get started, making it easy to use in for example CMS services.

So how is this functionality achieved?

### Setting the stage

For developing the Web Component, I wanted to stay as Vanilla as possible, making Rough Notation the only dependency in the project.

To make the development a bit easier, I also added [es-dev-server](https://www.npmjs.com/package/es-dev-server) as a Dev Dependency.
Es Dev Server enabled us to have a buildless environment with hot reloads.

At release the `package.json` looked something along the lines of:

```json
{
    "name": "vanilla-rough-notation",
    "version": "0.4.2",
    "description": "A vanilla implementation of the Rough Notation library",
    "main": "index.js",
    "module": "index.js",
    "type": "module",
    "files": ["*.js"],
    "scripts": {
        "start": "es-dev-server --app-index index.html --node-resolve --watch --open"
    },
    "author": "Matsuuu",
    "license": "MIT",
    "devDependencies": {
        "es-dev-server": "^1.54.0"
    },
    "dependencies": {
        "rough-notation": "^0.4.0"
    }
}
```

Next let's jump into the code itself.

### Writing the Web Component

First things we want to do are

-   Import the library we are porting
-   Create a Class containing our code
-   Make our class extend the HTMLElement class
-   Declare the Web Component as a custom element

Once we've done this, the code should look like this

```js
import { annotate } from 'rough-notation';

export default class VanillaRoughNotation extends HTMLElement {}

if (!customElements.get('rough-notation')) {
    customElements.define('rough-notation', VanillaRoughNotation);
}
```

Now if we used the html tag `<rough-notation></rough-notation>`, the code inside our class would be executed, if there was any.

Let's add some initial settings next.

### Let's start shadow boxing

One of the big pro's of Web Components are encapsulation of styles and code. This is done by attaching a ShadowRoot to the Web Component.

A [Shadow Root](https://developer.mozilla.org/en-US/docs/Web/API/ShadowRoot) is a part of the Shadow DOM API, and functions as a sort of subtree of the main DOM tree.
The Shadow Root is rendered separately from the document's main DOM tree, and is unaffected by the styles of the main DOM tree.

Using the Shadow DOM is necessary in our component since on top of creating a encapsulated environment, the Shadow DOM enables the use of [Slots](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/slot).

Slots are relevant part of the Web Components tech suite. They function as a placeholder inside the web component that can be filled with markdown.

Let's start our Web Component codebase by creating a shadow root and appending a slot inside it.

```js
export default class VanillaRoughNotation extends HTMLElement {
    connectedCallback() {
        if (!this.shadowRoot) {
            this.attachShadow({ mode: 'open' });
        }
        this.shadowRoot.innerHTML = '<slot></slot>';
    }
}
```

The `connectedCallback` method is part of the HTMLElement API we are extending.
`connectedCallback` is called each time the custom element is appended into a document-connected element

Inside the function we check if the element already has a shadowRoot, and in a case that a shadowRoot is missing, we attach one with `this.attachShadow({mode: 'open'})`

By setting the mode as 'open', we make our element accessible from Javascript outside the shadow root.
The contents of our element could be queried with `document.querySelector("rough-notation").shadowRoot`

If we set the mode as 'closed', we effectively deny all access to the nodes inside the shadow root from outside.

We also set the `<slot>` element as the body of the shadowRoot. Now we can add other DOM elements inside the rough-notation html tags and they will be shown with the component.

Allright. Now we can put elements inside our Web Component. Let's start implementing the library's functionality.

### Locked and noded

To implement the features of the API we went through earlier, we first need a way to get all of the elements inside our Web Component.

This can be done by querying the assigned nodes of our slot element.

```js
const assignedNodes = this.shadowRoot
    .querySelector('slot')
    .assignedNodes()
    .filter((node) => node instanceof HTMLElement);
```

When querying the assignedNodes, we might also catch some `#text` elements, since line breaks generate text nodes. We want to filter those out and can be
easily done by just filtering the nodes to instances of HTMLElement.

Next up we want to apply the library's functionality to all of the nodes. Different to normally calling the API, in a Web Component we need to take all
of the possible variables into account.

```js
assignedNodes.forEach((an) => {
    this.annotation = annotate(an, {
        type: this.type,
        animate: this.animate,
        animationDuration: this.animationDuration,
        color: this.color,
        strokeWidth: this.strokeWidth,
        padding: this.padding,
        multiline: this.multiline,
        iterations: this.iterations,
    });
});
```

---

#### Attributes and properties

But wait, you might ask. We haven't declared these variables yet. That's true. Next we will look at setting the properties.

The base values of a Web Component's properties are set in the `constructor` method of the element.

```js
 constructor() {
    super();
    this.type = 'underline';
    this.animation = true;
    this.animationDuration = 800;
    this.color = 'currentColor';
    this.strokeWidth = 1;
    this.padding = 5;
    this.showOnLoad = false;
    this.order = 0;
    this.multiline = true;
    this.iterations = 2;

    this.annotation = null;
}
```

Here we can either use the defaults of the library, or declare a default case of our choice. I decided to implement the default
values from the library, but also set `underline` as the default type of notation.

Currently we are rocking with the default values, but we of course want the user to be able to declare the attributes themselves.

Let's create a method for setting the variables from the HTML element attributes.

```js
setAttributes() {
    this.type = this.getAttribute('type') || this.type;
    this.animation = this.hasAttribute('animation') ? this.getAttribute('animation') === 'true' : this.animation;
    this.animationDuration = this.getAttribute('animationDuration') || this.animationDuration;
    this.color = this.getAttribute('color') || this.color;
    this.strokeWidth = this.getAttribute('strokeWidth') || this.strokeWidth;
    this.padding = this.getAttribute('padding') || this.padding;
    this.showOnLoad = this.hasAttribute('showOnLoad');
    this.order = this.getAttribute('order') || this.order;
    this.multiline = this.hasAttribute('multiline') ? this.getAttribute('multiline') === 'true' : this.multiline;
    this.brackets = this.getBrackets();
    this.iterations = this.getAttribute('iterations') || this.iterations;
}
```

Normal attributes we can fetched by just using the `this.getAttribute(attr)` method. But we also have a couple of boolean properties in our component.
These are a bit more trickier, since the HTML attributes can only be strings. In those cases we need to check for the existence of the attribute.

This can be done with `this.hasAttribute(attr)`. So now we can use the boolean operators by omitting the value part of the attribute like so:

```html
<rough-notation showOnLoad></rough-notation>
```

But if the default value of the property is true, this won't work. In these cases we want to check if the attribute is a string valued `true`.
So in this case if we want to disable the animation, we can just write.

```html
<rough-notation animation="false"></rough-notation>
```

If we wanted to observe the changes inside these properties, we could employ the use of `attributeChangedCallback` function and
custom setters for our properties. However our Web Component's values are set in the element initialization itself so this
won't be needed here.

---

#### Back on track

Now let's get back to our initialization code.

As you might have noticed, we added a new attribute to the API: `showOnLoad`. This can be used to easily enable the animation to show
as soon as it's ready, instead of running the animation when the `show()` method is called.
Now let's implement the functionality:

The Rough Notation library appends a style element into the main DOM, which has some crucial animation keyframes we want to make sure are
applied to our slotted elements too. Remember: Shadow Root encapsulates styles, so if our slotted elements are inside a shadow Root, the styles from
the main DOM won't affect them.

A fast tour through the source code of the library shows that the style element is assigned to a global variable `__rno_kf_s`.
Short for "Rough Notation Keyframe styles".

We can just clone that node inside our element:

```js
this.append(window.__rno_kf_s.cloneNode(true));
```

We want to make sure we clone it, and not just yank it from the main DOM.

Now if we immediately call `show()` after cloning the node everything should work, right?

But now we notice that the animation doesn't seem to play. What gives?

If we take a look at the Styles we want to clone, we notice that it's a keyframes styling along the lines of:

```css
@keyframes rough-notation-dash {
    to {
        stroke-dashoffset: 0;
    }
}
```

This means that this gives our library elements an end state for their animation. But with how Javascript works, by just cloning the node and
immediately calling the show method, the style element hasn't had time to initialize itself in the DOM, meaning that it's styles won't apply just yet.

One of the first solutions you might think of could be

> But if I'll just add a `setTimeout` that should fix it, right?

Well.. Yes, but no.

SetTimeout is all sorts of yucky and shouldn't be overused for situations like these. It's true that it would fix the problem but it might
introduce some new bugs into our component.

Instead we can just tell our code to wait for _the next_ animation frame, and then run the `show()` command. This should fix our issue.

To do that, we just write

```js
window.requestAnimationFrame(() => {
    if (this.showOnLoad) {
        this.annotation.show();
    }
});
```

The `requestAnimationFrame` takes a callback as an parameter, which it then calls after a frame has been shipped by the browser.

---

#### Closing in on the target

So right now we should have a somewhat functional port of the Web Component. Our source code looks something along the lines of:

```js
import { annotate } from 'rough-notation';

export default class VanillaRoughNotation extends HTMLElement {
    constructor() {
        super();
        this.type = 'underline';
        this.animation = true;
        this.animationDuration = 800;
        this.color = 'currentColor';
        this.strokeWidth = 1;
        this.padding = 5;
        this.showOnLoad = false;
        this.order = 0;
        this.multiline = true;
        this.iterations = 2;

        this.annotation = null;
    }

    setAttributes() {
        this.type = this.getAttribute('type') || this.type;
        this.animation = this.hasAttribute('animation') ? this.getAttribute('animation') === 'true' : this.animation;
        this.animationDuration = this.getAttribute('animationDuration') || this.animationDuration;
        this.color = this.getAttribute('color') || this.color;
        this.strokeWidth = this.getAttribute('strokeWidth') || this.strokeWidth;
        this.padding = this.getAttribute('padding') || this.padding;
        this.showOnLoad = this.hasAttribute('showOnLoad');
        this.order = this.getAttribute('order') || this.order;
        this.multiline = this.hasAttribute('multiline') ? this.getAttribute('multiline') === 'true' : this.multiline;
        this.iterations = this.getAttribute('iterations') || this.iterations;
    }

    connectedCallback() {
        this.setAttributes();
        if (!this.shadowRoot) {
            this.attachShadow({ mode: 'open' });
        }
        this.shadowRoot.innerHTML = '<slot></slot>';
        const assignedNodes = this.shadowRoot
            .querySelector('slot')
            .assignedNodes()
            .filter((node) => node instanceof HTMLElement);

        assignedNodes.forEach((an) => {
            this.annotation = annotate(an, {
                type: this.type,
                animate: this.animate,
                animationDuration: this.animationDuration,
                color: this.color,
                strokeWidth: this.strokeWidth,
                padding: this.padding,
                brackets: this.brackets,
                multiline: this.multiline,
                iterations: this.iterations,
            });
        });
        // Clone the style element from the windows styles to shadow dom.
        this.append(window.__rno_kf_s.cloneNode(true));

        window.requestAnimationFrame(() => {
            if (this.showOnLoad) {
                this.annotation.show();
            }
        });
    }
}

if (!customElements.get('rough-notation')) {
    customElements.define('rough-notation', VanillaRoughNotation);
}
```

Now the finishing touches before we ship this.

---

### Exposing the API

As you might have noticed, the Rough Notation library exposes a set functions for us to call. We want to enable our users to use
these functions as well. Luckily we can just expose those API's the same way that the original library does.

We see that there are 4 main functions the library has:

-   show()
-   hide()
-   remove()
-   isShowing()

To expose these API's we can just create wrapper functions for those inside our component:

```js
isShowing() {
    return this.annotation != null && this.annotation.isShowing();
}

show() {
    if (this.annotation) {
        this.annotation.show();
    }
}

hide() {
    if (this.annotation) {
        this.annotation.hide();
    }
}

remove() {
    if (this.annotation) {
        this.annotation.remove();
    }
}
```

Now our users can just call the functions by selecting our web components from the DOM and calling it.

```js
document.querySelector('rough-notation').show();
```

---

### Wrapping things up

Now we should have a functional Web Component we can use in our projects, no matter the framework.

The best thing about pure Vanilla Web Components are that they are framework agnostic, and don't rely on for example
React or LitElement to be imported into the project, making them really just Plug-and-Play.

Of course in this case, we are still relying on the rough notation libary, but there are plenty of Web Components built without
any dependencies.

Porting existing libraries is a great way to get into the feel of writing Web Components. They also make the world a little bit
simpler at the same time, since there is no more need for all that calling of library API's since the component does it already.

---

##### Links

-   [Original Library](https://github.com/rough-stuff/rough-notation)
-   [Web Component Port](https://github.com/Matsuuu/vanilla-rough-notation)
-   [MDN Resource on Web Components](https://developer.mozilla.org/en-US/docs/Web/Web_Components/Using_custom_elements)
-   [ES Dev Server](https://www.npmjs.com/package/es-dev-server)
