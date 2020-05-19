---
title: 'How I Stopped Being Scared and Started Loving Vim'
date: 2020-05-19T15:48:32+03:00
toc: false
images:
tags:
    - Vim
    - DX
    - Editors
---

Vim. It's the editor in the center of all freshman Computer Science student memes.

## ![Scientist meme vim](/scientist-vim-meme.png)

![Galaxy brain vim meme](/galaxy-brain-vim-meme.png)

It's something almost every developer has seen at some part of their career.
The most usual first touch to Vim is when new developers start using Git from
the command line, and there is a merge conflict, or you forget to type a commit message.

![Vim in git commit screen](/git-commit-vim.png)

> Oh no what is this how do I escape `<esc>` `<esc>` `<CTRL-C>`

And that's when you learn the holy combination of keys:

```bash
ESC
:wq
Enter
```

And just like that, you've exited Vim, and are on your way to change the default
editor to nano. ( Don't worry, I did that too )

## Adoption

A lot of programmers these days use IDE's or text editors depending on the language
and according to the [Stack Overflow Developer Survery in 2018](https://insights.stackoverflow.com/survey/2018/#technology-_-most-popular-development-environments),
Vim isn barely in the top 5.

A lot of people have moved from heavier IDE's to fast editors with plugins like
[VS Code](https://code.visualstudio.com/) and [Atom](https://atom.io/).

For more demanding tasks or just for people who want a better tooling from their
editor, products of [JetBrains](https://www.jetbrains.com/) have been popular.

I used to use the JetBrains product family for all of my projects.
Webstorm for Javascript, IntelliJ for Java, Rider for Unity etc.
The problem with IDE's is that them beeing beefier than regular editors,
their startup time and system overhead are also usually higher.
On lower end pc's this might mean opening the IDE and leaving to grab a coffee
while the editor starts.

> So if these editors already provide the necessary tooling, why would I want to move to Vim?

### Vim Users

Vim users are usually something you don't see too much in the wild.
VSCode supporting more and more languages by the day due to numerous plugins
has been taking over the ecosystem, wether you're writing C or Javascript.

But when you see one, typing away away in _just a terminal window_, it can be
sort of a confusing sight. That's mainly because of the customizability
of Vim. Most Vim users configure their editor to **work** and **look** the way _they_ want it to,
as opposed to for example VSCode, in which a lot of users go with the default settings.

Another difference between Vim and other popular editors is that vim lacks a proper GUI.
When you open up VSCode, you are greeted with a nice file tree and action dropdowns
in the top bar. When you open Vim, you are greeted with ~~pain and agony~~ either a
basic file tree or just a text file. And worst of all, you can't click anywhere.

### So what makes Vim such a good editor

Many people who have not used Vim, or have had short ettempts at using Vim usually
dislike Vim, but people who have used it for multiple months or years usually praise it,
why could this be? What are the benefits of Vim?

---

### Using Vim: The performance perspective

I'll try to remain objective here, but some of the main benefits, without going to the
actual usage include:

-   It's already installed in most operating systems

    -   Vim comes preinstalled on almost all popular Linux distributions and also on OSX.
    -   Whenever SSHing to a new server on a cloud provider, the editor installed is usually vim.

-   It runs in the terminal, causing close to no overhead on the system.

    -   Vim is extremely lightweight, and will run on even the oldest of laptops.

-   Vim utilizes the keyboard
    -   Vim uses the keyboard for all of it's actions, meaning you don't have to switch
        between the mouse and keyboard all of the time.

These are some of the great things about having vim as your editor.
How about the pro's of Vim from the perspective of the user?

---

#### Using Vim: The developer experience

![Vim learning curve](https://i.stack.imgur.com/7Cu9Z.jpg)

The learning curve of Vim has to be the biggest reason of a smaller adoption of the editor.
It is true, that Vim can be hard to learn, but as soon as you learn the "language" of Vim and
some of the basic commands, you are on your way to a greater development experience.

Learning Vim is a skill that helps with a lot of applications, especially if working with Linux, since
you learn some of the basic commands of Unix systems along the way.

Wherein learning a normal text editor, some of it's shortcuts and gimmicks might take an afternoon,
learning Vim is a bit lengthier of a process, but I promise you, it's worth it.

A quote in a Reddit thread discussing Vim described the process well:

> Vi/Vim echoes the design philosophy of Unix.
> In both Unix and in Vi, you have a collection of atomic commands, which perform one task.
> More complicated tasks are done by combining the smaller predefined tasks.

And example of this could be combining two simple commands:

Say a user wants to empty a file of its contents. In a normal editor the easiest way would be to select all text,
and then remove it by pressing delete or the backspace.

In Vim you would use 2 commands for this:

-   The `d` command to tell Vim you're about to delete something
-   the `G` command, to tell Vim to go to the last line of the file.

These commands combined will tell Vim to "Delete everything from my current position until the last line of the file".

Nothing too out of the ordinary here, but let's get into some more powerful command combinations:

---

#### Using Vim: The speed

The speed of Vim comes when you learn some of the basic commands, and start using them in conjunction with each other.

Let's look at a few use cases.

In Vim, you can use the `w` key to move to the start of the next word. This functionality is fimiliar to the one of
using CTRL + Right arrow on your keyboard, but it can be done with just one keystroke.
Moving backwards is done by pressing the `b` key. These can be memorized by "Move to the next **W**ord" and "Move to the word **B**efore"

Now let's see them in action.

Say we wanted to move forwards 10 words. In normal situations we would need to hold CTRL and type Right arrow 10 times.
With Vim we only need to tell Vim to execute the `w` command 10 times with `10w`. Now that's handy!

Now let's say we wanted to change the word our cursor is currently at the start of. In a normal editor. we would again need to paint the word,
and type something on top of it, or even worse: delete the word letter by letter. In Vim, we can issue 2 commands.

-   The `c` command, which acts almost the same to the `d` command, but **C**hanges the content instead of **D**eleting it
-   The `w` command we already discussed.

What this will do is remove the word our cursor is currently at the start of, and go into Insert mode, in which you do your typing in Vim.

This can also be combined with the movement we learned earlier. If we type `c3w`, we are able to replace the next 3 words.

> Okay okay those are some neat tricks, but I don't see the time save in these yet.

Then it's time for the **even more powerful** vim commands.

---

#### Using Vim: The extreme speed

Say we're in a situation, in which we need to remove all of the content inside curly braces ({}).

On a normal text editor, you would most likely just paint the contents with a mouse, and then delete.
Trying not to paint the opening or closing brace while you're at it

In Vim, this kind of editing is what Vim shines in. There are determined boundaries and rules.

To delete everything inside the curly braces in Vim, you only need to position your cursor inside the curly braces. Can be anywhere really.

Then you just type `di{` or `di}`, whichever is fine.

What this will do is it tells Vim to "**D**elete everything **I**nside the boundary", in which the boundary is `{`.

Now that's real speed. From multiple movements to just 3 keystrokes.

---

### The Vim Language

As you might have noticed, I usually wrote the Vim commands as "What I tell Vim to do".
That's something that Vim users call the Vim Language. Memorizing the shortcuts and commands by
issuing meaning and "motion" to the words.

A great talk about the Vim language can be found [here](https://www.youtube.com/watch?v=wlR5gYd6um0)

---

### The verdict

I only scratched the surface of how movement commands and basic text editing work in Vim, but there's so much more
under the surface. Vim being open source, and fully customizable, the limits are endless on what you can do with it.

I started using Vim about a month ago, and it's now my daily driver as a code editor. With the help of a few plugins, I'm
getting the same, if not even better developer experience as I would get from using for example VSCode.

On top of that, the vim commands provide me with big a huge utility of tools to use for editing, navigating and formatting my code.

So if you're on the edge of checking out Vim, I urge you to try it out. You can do so easily on any device supporting vim by typing `vimtutor` in the terminal.

For any questions about my configurations or comments, you can DM me in twitter.
