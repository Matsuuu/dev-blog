---
title: "Getting Hooked on Stoxy"
date: 2021-04-22T19:50:38+02:00
---

Stoxy is a modern state management library built around creating reactive, stateful and persistent web experiences.

Stoxy allows you to easily control the global state of your application, and tap into said state when needed.

The newest addition to Stoxy is a new add-on library: **Stoxy Hooks**. 

Stoxy Hooks are a easy way to integrate Stoxy to any **React** or **Preact** application.

### Examples

Here I'll show a few simple examples of Stoxy Hooks in action

#### A Simple Clicker

```jsx
import { useStoxy } from "@stoxy/hooks";
import React from "react";

export function Clicker() {
    const { state, update } = useStoxy(React, {
        key: "demo.counter",
        state: 0
    });

    function inc() {
        update(c => c += 1);
    }

    return (
        <div>
          <p>Pushed {state} times</p>
          <button onClick={inc} type="button">Click</button>
        </div>
    );
}
```

#### A Todo List


```jsx
import { useStoxy } from "@stoxy/hooks";
import * as preact from "preact/hooks";
export function TodoList() {
    const { state } = useStoxy(preact, {
        key: "todo-list",
        state: {
            items: []
        },
        init: true,
        persist: true
    });

    return (
        <ul>
            {state.items.map(item => <li key={item.id}>{item.name}</li>)}
        </ul>
    );
}
```

```jsx
import { useStoxy } from '@stoxy/hooks';
import React from 'react';

export function AddToList() {
    const { add } = useStoxy(React, { key: 'todo-list' });

    function addItem(e) {
        e.preventDefault();
        const formData = new FormData(e.target);
        const taskName = formData.get('task');

        add({ created: Date.now(), name: taskName });

        const inputField = document.querySelector("input[name='task']")
        inputField.value = "";
    }

    return (
        <form onSubmit={addItem}>
            <input type="text" name="task" />
            <input type="submit" value="Add" />
        </form>
    );
}
```

### Get Started

You can easily get started using Stoxy hooks with just one quick install:

```bash
npm install @stoxy/hooks
```

And you're all set!

The whole Stoxy ecosystem is extremely lightweight, in package size and when writing code.

Read more about the subject on the [Stoxy Website](https://stoxy.dev/)


If you like how Stoxy makes managing state simple, [Join the almost 50 Stargazers on GitHub](https://github.com/stoxy-js/stoxy/stargazers)
