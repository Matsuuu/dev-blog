---
title: "Re-Learning Clojure. Part one: Getting my feet wet"
date: 2021-08-08T23:26:00+02:00
---
After following my friend [Juhis's series on learning rust](https://hamatti.org/posts/learning-rust-pattern-matching/),
I decided I should pick up Clojure again. And while I'm at it, I want to revitalize my blog writing, so here we are: Two stones with one bird.

I first learned Clojure in 2018 while working for my previous employer and I really enjoyed my time with!

However work took me towards another path; developing Java applications with some Javascript, and then
moving to working more with Javascript. However, Clojure changed me. My Java(script) code was different after
dipping my toes into Clojure. I thought more in a functional mindset, utilizing more functional styles of working:
Using maps and reducers, writing more and more recursive functions, and most of all: Not mutating as much state as I did before.
I had grown tired of side effects, and approaching problems from a more functional point of view did help with that.

Now after a few years of developing mostly in OOP-inspired languages, I'm re-learning Clojure, and I'm doing it
the way a lot of developers did it: By following the book [Clojure for the Brave and True](https://www.braveclojure.com/clojure-for-the-brave-and-true/).

## Chapter One
[Link](https://www.braveclojure.com/getting-started/)


### Introduction 

To start off, I love the more laid back approach to learning a language the book provides. From the first chapter,
I was giggling away at the way this book lays stuff out.

I was also so happy to find out about the "4 Labyrinths". Those Labyrinths being:

- Tooling
- Language (syntax, semantics, data structures)
- Artifacts (building projects, distributing them, etc.)
- Mindset

Earlier in my career I tried learning C++ by doing, instead of reading a dry C++ book, and it was quite a slope.
I had trouble finding good material on how to approach getting my first programs to run, how I should structure my programs,
how Makefiles work etc.

It's a refreshing thing to know that from the get-go these things are getting sorted out.


### Getting started

Even though the beginning steps were familiar to me due to having worked with Clojure, I still enjoyed
being called a little teapot! I'll try to do all of the exercised even if they are obvious to me.

I was a bit hesitant as the author was bringing up Emacs. As a avid Vim user, I really didn't feel like
delving into emacs at this point. Luckily it's only optional, and the author even listed nice resources for other
editors too.

I had actually set up my dev environment for Clojure on Vim earlier while trying stuff out, so I was happy running with the
following setup:

- [The Clojure LSP](https://github.com/clojure-lsp/clojure-lsp)
- [Conjure](https://github.com/Olical/conjure)
- [NeoVim 0.5.x](https://neovim.io/)

And after reading through the Conjure bindings from `:help conjure-mappings`, I was all set. I had a REPL
that works with Vim, and i can evaluate forms with a simple mapping (space + ee) in my case.

I however still went through the emacs chapter out of interest. Know thy enemy and so on.

Yay! Chapter one has been cleared by this little teapot. Next onto Chapters 2 and 3.



## Chapter Two
[Link](https://www.braveclojure.com/basic-emacs/)

As this is the emacs chapter, I will mostly be scimming through it. I've always been interested in emacs,
but vim is currently my editor of choice and I'm not changing at the moment.

I really enjoy how in depth the book goes into learning emacs. This must be a great experience for someone 
picking up the editor and doing their journey through the book with it, learning emacs AND Clojure while they're at it.


## Chapter Three
[Link](https://www.braveclojure.com/do-things/)

### Syntax

I love how the book just jumps straight into action. We are promised fun, we get fun. We get to make hobbits
and hit them! What could be better.

I can't help but chuckle at the way this book is written. Everything from the tone of every sentence to the
parantheses mines beneath Massachusetts. As someone who doesn't really read books, this is some captivating writing.

I also have to add: I absolutely love the Panda with the Dust buster.

Allright let's get into the meat of things, the code.

I like how we get straight into things, and start working with the most basic (and important) language features:
iteration and conditionals. The addition of how-and-why to use the `do` keyword was extremely helpful too

As someone who works with both Javascript and Java, I enjoy that Clojure has the `truthy` and `falsey` logical 
representations. Too many times I've had to write `if (something == null)` in Java. No more!

As someone a bit familiar with Clojure, and familiar with Functional programming, it's clear to me, but it's really nice
that we are approaching immutability from the get-go and that comes out clearly from the use of the word "bind" as the author explains.

It's also nice to follow along the explanations, since every time a new keyword is thrown at us, we are also told _when_
we are told what it actually means, instead of just hoping that it gets explained to us at some point, or god forbid,
is expected for us to know.

### Data structures

When learning a new language, it's quite important to know about the data structures used by the language. Especially when working
with such a different beast like Clojure, it's really good to familiarize yourself with the new data structures and their names.

As I saw the first use of keywords, I was wondering why they were just scooted over, but here we are luckily told that it'll
be explained soon. From what I remember from my Clojure times, Keywords are quite a big part of development.

#### Maps

The maps were nicely explained but I found maybe a bit lacking on the explanation on why use `get` if you can just look up the value by omitting
it and using the map itself as the operator.

```clojure
({:name "The Human Coffeepot"} :name)
; vs
(get {:name "The Human Coffeepot"} :name)
```

#### Keywords

The next confusing step was that the function above was turned upside down and it still works

```clojure
(:a { :a 1 :b 2})
; => 1
```

Maybe that's just the magic of Clojure.

Well the line 

> Using a keyword as a function is pleasantly succinct, and Real Clojurists do it all the time. You should do it too!

made me trust that I should just go with the keyword approach for all of this. It's great to get a straight up recommendation.

#### Vectors and Lists

Going into the the next section, it's again really nice to get a affirmative recommendation from the author
on where to use vectors, and where to use lists. When learning a new language I seem to always struggle to find the 
best data structure for each case and it can become a burdain. This really eases that burdain.


#### Sets

The quick scim through sets was plenty enough for someone who know the jist of how sets work in other languages.

Having examples of usage and also value retrieval from all of these data structures is extremely helpful.

I've been enjoying the simplicity of Clojure and how you can do so much with so little. I've actually watched a couple of
Clojure talks by Rich Hickey himself, and grown to love how you can do so much with such a small set of language details to remember.
It reminds me of the [Natural Semantic Metalanguage theory](https://en.wikipedia.org/wiki/Natural_semantic_metalanguage), in which you can
reduce a lexicon down to a set of semantic primitives, which you can use to explain any word in the language. For my finnish readers there's
a finnish version of it [called "65 Words" here](https://www.65sanaa.fi/).


#### Functions

Allright. That's enough about data structures, next we'll go into the meat of programming: Functions.

We just go straight into the crazy syntax of Lisps and how you can just create there horrid monsters like

```clojure
((and (= 1 1) +) 1 2 3)
; => 6

((first [+ 0]) 1 2 3)
; => 6
```
However, like watching a car crash in slow motion, I can't turn and look away. It's extremely intriguing how you can take
such a simple way of creating functions, and take it so far to create this kind of funky monsters. It makes you wonder
what kind of actually useful function you could create with the same approach.

The examples given about evaluation, and how the evaluation is done, step by step is really helpful to help understand
how to structure your code in Clojure and how to make the best use of it. Going into how some `special forms` bend the rules
a bit is something that is best done with examples, and those are provided.

As someone who tends to write a lot of JavaDoc and JSDoc to explain what's going on, I appriciate the time the author put in
to actually highlight the DocString as a equally important part of the function in the "Defining Functions" -section.


##### Arity overloading

I had completely forgotten about this gem. This is one of the cool things that got me into Clojure, and I really enjoy
how simple it's made in Clojure to do this.

Simply put, you can handle the function in different ways depending on the number of args given e.g.

```clojure
(defn x-chop
  "Describe the kind of chop you're inflicting on someone"
  ([name chop-type]
     (str "I " chop-type " chop " name "! Take that!"))
  ([name]
     (x-chop name "karate")))
```

Working with Java, I often run into situations where I'd like to do this, and I often do, but the syntax to achieve it is so much more cumbersome.

```java
void doThing(WayOfDoing way) {
    // Apply way on thing
}

void doThing() {
    doThing(WayOfDoing.DEFAULT_WAY);
}
```

#### Destructuring

Ah. A familiar term from JavaScript land. This should be a nice refresher on destructuring objects.

Finding out that you can destructure lists and vectors was expected, but the use of the rest parameter here is a nice 
reminder for developers that you can do that too.

Destructuring maps is quite cool, as it kind of works in the same premise as fetching a value by keyword from a map.
The syntax however is something I really need to freshen my memory on. I can't seem to remember when to add what kinds of 
parentheses around the function arguments.

Example given by the book on map destructuring:

```clojure
(defn announce-treasure-location
  [{lat :lat lng :lng}]
  (println (str "Treasure lat: " lat))
  (println (str "Treasure lng: " lng)))

(announce-treasure-location {:lat 28.22 :lng 81.33})
; => Treasure lat: 28.22
; => Treasure lng: 81.33
```

What really wooed me over was the `:keys` -keyword you can use in your function parameter declaration. It's so simple, yet so effective!

```clojure
(defn announce-treasure-location
  [{:keys [lat lng]}]
  (println (str "Treasure lat: " lat))
  (println (str "Treasure lng: " lng)))
```


The `:as` keyword also shows that all of these cases have really been thought through. Sometimes you want to have the full object AND
destructure some of the values, especially when working in functional languages.

#### Anonymous functions

Oh boy. I've been working so much with Javascript and Java Lambdas that I've grown extremely accustomed to anonymous functions.
I can remember however, in 2018 when learning Clojure, I was so weirded out about "Why would you use anonymous functions". Oh silly 2018 me.

Reader functions are something that I really need to look into. They look weird and I don't really understand them.

Edit: Okay yeah Matsuu from 5 minutes into the future. The explanation on how to use the Reader functions is quite good
and I can imagine using them has their place. However I think I'll not be littering my codebase with those yet.


### Ending words

Allright. We got through the teaching part of Chapter 3. As it's 23.25 over here and I've got work tomorrow, I'll be leaving the coding part
for next time.

I'll be documenting my learnings in this blog as I go forwards as writing them out helps me internalize the information better myself.

Until next time! And remember to be brave!

<script>
setTimeout(() => {
    Prism.languages.clojure = { comment: /;.*/, string: { pattern: /"(?:[^"\\]|\\.)*"/, greedy: !0 }, operator: /(?:::|[:|'])\b[a-z][\w*+!?-]*\b/i, keyword: { pattern: /([^\w+*'?-])(?:def|if|do|let|\.\.|quote|var|->>|->|fn|loop|recur|throw|try|monitor-enter|\.|new|set!|def-|defn|defn-|defmacro|defmulti|defmethod|defstruct|defonce|declare|definline|definterface|defprotocol|==|defrecord|>=|deftype|<=|defproject|ns|\*|\+|-|\/|<|=|>|accessor|agent|agent-errors|aget|alength|all-ns|alter|and|append-child|apply|array-map|aset|aset-boolean|aset-byte|aset-char|aset-double|aset-float|aset-int|aset-long|aset-short|assert|assoc|await|await-for|bean|binding|bit-and|bit-not|bit-or|bit-shift-left|bit-shift-right|bit-xor|boolean|branch\?|butlast|byte|cast|char|children|class|clear-agent-errors|comment|commute|comp|comparator|complement|concat|conj|cons|constantly|cond|if-not|construct-proxy|contains\?|count|create-ns|create-struct|cycle|dec|deref|difference|disj|dissoc|distinct|doall|doc|dorun|doseq|dosync|dotimes|doto|double|down|drop|drop-while|edit|end\?|ensure|eval|every\?|false\?|ffirst|file-seq|filter|find|find-doc|find-ns|find-var|first|float|flush|for|fnseq|frest|gensym|get-proxy-class|get|hash-map|hash-set|identical\?|identity|if-let|import|in-ns|inc|index|insert-child|insert-left|insert-right|inspect-table|inspect-tree|instance\?|int|interleave|intersection|into|into-array|iterate|join|key|keys|keyword|keyword\?|last|lazy-cat|lazy-cons|left|lefts|line-seq|list\*|list|load|load-file|locking|long|macroexpand|macroexpand-1|make-array|make-node|map|map-invert|map\?|mapcat|max|max-key|memfn|merge|merge-with|meta|min|min-key|name|namespace|neg\?|newline|next|nil\?|node|not|not-any\?|not-every\?|not=|ns-imports|ns-interns|ns-map|ns-name|ns-publics|ns-refers|ns-resolve|ns-unmap|nth|nthrest|or|parse|partial|path|peek|pop|pos\?|pr|pr-str|print|print-str|println|println-str|prn|prn-str|project|proxy|proxy-mappings|quot|rand|rand-int|range|re-find|re-groups|re-matcher|re-matches|re-pattern|re-seq|read|read-line|reduce|ref|ref-set|refer|rem|remove|remove-method|remove-ns|rename|rename-keys|repeat|replace|replicate|resolve|rest|resultset-seq|reverse|rfirst|right|rights|root|rrest|rseq|second|select|select-keys|send|send-off|seq|seq-zip|seq\?|set|short|slurp|some|sort|sort-by|sorted-map|sorted-map-by|sorted-set|special-symbol\?|split-at|split-with|str|string\?|struct|struct-map|subs|subvec|symbol|symbol\?|sync|take|take-nth|take-while|test|time|to-array|to-array-2d|tree-seq|true\?|union|up|update-proxy|val|vals|var-get|var-set|var\?|vector|vector-zip|vector\?|when|when-first|when-let|when-not|with-local-vars|with-meta|with-open|with-out-str|xml-seq|xml-zip|zero\?|zipmap|zipper)(?=[^\w+*'?-])/, lookbehind: !0 }, boolean: /\b(?:true|false|nil)\b/, number: /\b[\da-f]+\b/i, punctuation: /[{}\[\](),]/ };
    Prism.highlightAll();
});
</script>

