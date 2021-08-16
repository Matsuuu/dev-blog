---
title: "Re-Learning Clojure. Part two: High octane action"
date: 2021-08-11T19:00:00+02:00
---

Re-Learning Clojure is a blog series of my adventures through [Clojure for te Brave and True](https://www.braveclojure.com/). 

In this series I'm trying to re-learn clojure and deepen my skills in the language.

## Putting it all together

[Link](https://www.braveclojure.com/do-things/#Pulling_It_All_Together)

Let's get straight into the action, continuing from where we left off.

After fighting a bit with some Vim autocommands, I got [auto-pairs](https://github.com/jiangmiao/auto-pairs) working. With Clojure I feel
it's beneficial to have the brackets close themselves by default. I don't really enjoy this in other languages but here it's 
something I'm at least giving a try.

Some of you might be interested in what my development setup looks like.

Here's a quick webp to showcase what my editor looks like while writing some of the more basic examples.

![Clojure Dev](/clojure-dev.webp)

I change up where I set up the REPL output, but currently it's on the right side as a log file. Some of the REPL actions also get 
displayed inline with the code. This is achieved with Conjure.

And the setup is also easy: Just run `lein repl` in one tmux window, and open up vim in another.

I could set it up so that the repl runs when I open up Vim but I don't really want to bind that kind of
things into my vim config.


---

#### So many parantheses to remember

Going through the usages of `let`, a kind of binding, I already know I'm going to be getting in trouble with
knowing how many sets of braces and parantheses I need to put into declarations.

Take this piece of code for example:

```clojure
(let [[first-dog & rest-of-the-dogs] dalmatian-list]
    [first-dog rest-of-the-dogs])   
; => ["Pongo" ("Perdita" "Puppy 1" "Puppy 2")]
```

The fact that you can assign multiple variables from a single list, and the amount of square brackets surrounding them
is going to be something I know I'll be fighting with.

--- 

#### Muscle memory is best memory

I'm trying out all of the code examples shown in the book, and trying to work on remembering the mappings used 
to evaluate forms, outer forms, visually selected forms etc. I'll try to improve my vim movement to match the demands
of Clojure, which are clearly different from the other languages I've written with.


### All the kinds of recursion

One of the big things I remember from Clojure is recursion. As we are not really allowed to have mutable variables, we need to utilize
recursion to iterate through changing values.

The first type of recursion we're introduced here is the `loop` - `recur` way of doing recursion. I like how clear it is once you
realize how the work in tandem.

```clojure
(loop [iteration 0]
  (println (str "Iteration " iteration))
  (if (> iteration 3)
    (println "Goodbye!")
    (recur (inc iteration))))
; => Iteration 0
; => Iteration 1
; => Iteration 2
; => Iteration 3
; => Iteration 4
; => Goodbye!
```

### Matchy matchies.

I like how simple the regexes are made in Clojure. Coming from Java, with Pattern and Match classes, and Javascript, with RegExp -classes, I like how you 
can just put a `#` before a string and now it's a Regex!

```clojure
; Find if the string "left-eye" starts with "left-" 
(re-find #"^left-" "left-eye")
; => "left-"

```


<script>
setTimeout(() => {
    Prism.languages.clojure = { comment: /;.*/, string: { pattern: /"(?:[^"\\]|\\.)*"/, greedy: !0 }, operator: /(?:::|[:|'])\b[a-z][\w*+!?-]*\b/i, keyword: { pattern: /([^\w+*'?-])(?:def|if|do|let|\.\.|quote|var|->>|->|fn|loop|recur|throw|try|monitor-enter|\.|new|set!|def-|defn|defn-|defmacro|defmulti|defmethod|defstruct|defonce|declare|definline|definterface|defprotocol|==|defrecord|>=|deftype|<=|defproject|ns|\*|\+|-|\/|<|=|>|accessor|agent|agent-errors|aget|alength|all-ns|alter|and|append-child|apply|array-map|aset|aset-boolean|aset-byte|aset-char|aset-double|aset-float|aset-int|aset-long|aset-short|assert|assoc|await|await-for|bean|binding|bit-and|bit-not|bit-or|bit-shift-left|bit-shift-right|bit-xor|boolean|branch\?|butlast|byte|cast|char|children|class|clear-agent-errors|comment|commute|comp|comparator|complement|concat|conj|cons|constantly|cond|if-not|construct-proxy|contains\?|count|create-ns|create-struct|cycle|dec|deref|difference|disj|dissoc|distinct|doall|doc|dorun|doseq|dosync|dotimes|doto|double|down|drop|drop-while|edit|end\?|ensure|eval|every\?|false\?|ffirst|file-seq|filter|find|find-doc|find-ns|find-var|first|float|flush|for|fnseq|frest|gensym|get-proxy-class|get|hash-map|hash-set|identical\?|identity|if-let|import|in-ns|inc|index|insert-child|insert-left|insert-right|inspect-table|inspect-tree|instance\?|int|interleave|intersection|into|into-array|iterate|join|key|keys|keyword|keyword\?|last|lazy-cat|lazy-cons|left|lefts|line-seq|list\*|list|load|load-file|locking|long|macroexpand|macroexpand-1|make-array|make-node|map|map-invert|map\?|mapcat|max|max-key|memfn|merge|merge-with|meta|min|min-key|name|namespace|neg\?|newline|next|nil\?|node|not|not-any\?|not-every\?|not=|ns-imports|ns-interns|ns-map|ns-name|ns-publics|ns-refers|ns-resolve|ns-unmap|nth|nthrest|or|parse|partial|path|peek|pop|pos\?|pr|pr-str|print|print-str|println|println-str|prn|prn-str|project|proxy|proxy-mappings|quot|rand|rand-int|range|re-find|re-groups|re-matcher|re-matches|re-pattern|re-seq|read|read-line|reduce|ref|ref-set|refer|rem|remove|remove-method|remove-ns|rename|rename-keys|repeat|replace|replicate|resolve|rest|resultset-seq|reverse|rfirst|right|rights|root|rrest|rseq|second|select|select-keys|send|send-off|seq|seq-zip|seq\?|set|short|slurp|some|sort|sort-by|sorted-map|sorted-map-by|sorted-set|special-symbol\?|split-at|split-with|str|string\?|struct|struct-map|subs|subvec|symbol|symbol\?|sync|take|take-nth|take-while|test|time|to-array|to-array-2d|tree-seq|true\?|union|up|update-proxy|val|vals|var-get|var-set|var\?|vector|vector-zip|vector\?|when|when-first|when-let|when-not|with-local-vars|with-meta|with-open|with-out-str|xml-seq|xml-zip|zero\?|zipmap|zipper)(?=[^\w+*'?-])/, lookbehind: !0 }, boolean: /\b(?:true|false|nil)\b/, number: /\b[\da-f]+\b/i, punctuation: /[{}\[\](),]/ };
    Prism.highlightAll();
});
</script>

