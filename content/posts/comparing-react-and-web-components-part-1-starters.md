---
title: 'Comparing React and Web Components. Part 1: Starters'
date: 2020-07-31T13:00:00+03:00
---

I've always shunned away from big frameworks while working with Javascript.
I used to work with React, but after gaining more experience with the language, I've started to go closer to vanilla with
everything I do.

### Comparing starter scripts

I ran the recommended starters for both React and Lit Element, one of the most used Web Component libraries.

For React, the started included 262 mb of depndencies just to get started

```bash
npm create-react-app react-example
cd react-example
du -sh node_modules
>> 262M
```

For Lit Element, I used the OpenWC recommended starter

```bash
npm init @open-wc
cd lit-element-example
du -sh node_modules
>> 161M
```

That's just a tad bit over 100 MB worth of code more to get started with building some web applications.

Not to talk about the dependency tree. Create-React-App seems to not make use of the devDependencies tag of the package.json.
All of the testing packages are included as normal dependencies.

```js
"dependencies": {
    "@testing-library/jest-dom": "^4.2.4",
    "@testing-library/react": "^9.3.2",
    "@testing-library/user-event": "^7.1.2",
    "react": "^16.13.1",
    "react-dom": "^16.13.1",
    "react-scripts": "3.4.1"
}
```

This won't cause any problems most of the time, but listing the deps clearly used only in development as devDependencies would be
welcome.

Most of React's libraries are low on dependencies, only containing in the ballpark of 2-6 deps. But then we look at the `react-scripts`
included in the create-react-app and it contains a whopping 53 dependencies! That's 11 times more dependencies to look after than the
other packages. With the Lodash fiasco we just had, making everyone bump their packages, having that many dependencies sure does make
the likelyhood of that happening again a lot larger.

If we look at the picture on the OpenWC side, the starter asks for what packages you want to include in your starter,
from packaging to testing to linting to demoing. For this purpose I chose everything but not demoing (storybook).

Our normal dependencies are looking great with just 2 libraries:

```js
  "dependencies": {
    "lit-html": "^1.0.0",
    "lit-element": "^2.0.1"
  }
```

And the only cross dependency here is that Lit-Element depends on lit-html, nothing else.

On top of that the starter has a lot more of useful packages included like linting with eslint, prettifying with prettier and husky for pre-commit hooks.
Here's the whole devDependencies tree or the OpenWC starter:

```js
  "devDependencies": {
    "eslint": "^6.1.0",
    "@open-wc/eslint-config": "^2.0.0",
    "prettier": "^2.0.4",
    "eslint-config-prettier": "^6.11.0",
    "husky": "^1.0.0",
    "lint-staged": "^10.0.0",
    "@open-wc/testing-karma": "^3.0.0",
    "deepmerge": "^3.2.0",
    "@open-wc/testing": "^2.0.0",
    "@open-wc/building-rollup": "^1.0.0",
    "rimraf": "^2.6.3",
    "rollup": "^2.3.4",
    "es-dev-server": "^1.5.0"
  },
```

Last we can check the build sizes both of the starters end up with.

I ran `npm run build` on both of these clean installations and then checked the size of the export folders.

For React, we are standing at a whopping 548K with the whole folder included and 472K inside the js folder.

For OpenWc, we end up with just 168K with the whole build folder, from which workbox seems to take most of.
If we won't be using workbox, we can delete these and our dist folder size drops down to just 40K.

That's quite a large size difference with just a starter level package. And considering a lot of
people use these starters as their basis, they might be shipping a lot more bloat than they think.

### Bottom line

While the create-react-app is a great tool, it brings a lot more bloat than with alternative frameworks.
While some of this might be able to be gotten rid of with some treeshaking, a lot of projects
don't use technologies that allow for that, and just ship the whole bundle as is.

With OpenWC, developers are able to get started with a more customizable starter build, and a lot less
unnecessary bloat included. The package size for a finished build can be almost 10 times smaller than when using React.

In the next post we'll be discussing developing for both React and Web Components, and the usability of
said developed components cross framework.
