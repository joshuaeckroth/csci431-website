---
title: Prolog resolution
layout: note
---

# Prolog resolution

Resolution is the process (in Prolog) of determining whether a **rule** can be proved. The process uses [unification](/notes/prolog-unification.html) to match variables and predicates, but the process known as resolution concerns the depth-first search process for finding variable assignments that satisfy rules (and the rules they refer to, etc. recursively).

## Resolution algorithm

Suppose we have the following Prolog knowledge base:

~~~ prolog
f(a). 
f(b). 

g(a). 
g(b). 

h(b). 

k(X) :- f(X), g(X), h(X).
~~~

How does Prolog determine whether `k(Y)` is true (provable)? That is,
how does Prolog determine the value for `Y` that makes the query true?

Its search process can be see in the graphic below. The process is as
follows:

1. Create a temporary variable `_G34` (randomly-named) to stand in
   for `Y`. This is an implementation detail, so that if some other
   rule uses `Y` (a completely different variable), then the
   variable names don't collide.

2. The goal is to prove `k(_G34)`. To do so, proving `f(_G34),
   g(_G34), h(_G34)` will be enough. This is the new goal.

3. To satisfy the first part of the new goal, `f(_G34)`, the
   knowledge base is searched. There is no rule for `f/1` (the
   predicate with arity one) but there is a fact: `f(a)`. *The first
   matching fact/rule is tried first.* Thus, `_G34` gets set to
   `a` (from unification).

4. Now, `g(a), h(a)` is the new goal. `g(a)` is satisfied just fine,
   because exactly that term is found in the knowledge base (trivial
   unification).

5. Now, `h(a)` is the new goal. But, nothing in the knowledge base
   unifies with `h(a)` and there is no `h/1` rule, so there is a
   problem.

6. Go to the last decision point. This was when `_G34` was set to
   `a`. Try to set it to something else. `f(b)` is in the knowledge
   base as well, so go with `_G34 = b`.

7. The new goal is `g(b), h(b)`, etc... (which ultimately works).

![Prolog resolution](/images/prolog-resolution-ex1.png)

## Another example

Now we'll switch to this knowledge base:

~~~ prolog
loves(vincent, mia).
loves(marcellus, mia).
jealous(A, B) :- loves(A, C), loves(B, C).
~~~

How does Prolog find the proof for `jealous(X, Y)`, and what values do
the variables take?

The proof tree is shown below. Note that there are four smiley-face
leaves in the tree. This means there are four different ways to get a
proof, which means there are four different sets of variable
assignments.

![Prolog resolution](/images/prolog-resolution-ex2.png)

## A list example

Finally, let's look at a list predicate.

~~~ prolog
member(X, [X|_]).
member(X, [_|T]) :- member(X, T).
~~~

This is familiar to us from the [Prolog](/notes/prolog.html)
notes. When we query with `member(X, [a, b, c])` we get that `X = a`.

![Prolog resolution](/images/prolog-resolution-ex3.png)

When we query with `member(d, [a, b, c])` we get failure.

![Prolog resolution](/images/prolog-resolution-ex4.png)

Much of these notes were adapted from
[Learn Prolog Now!](http://www.learnprolognow.org), a free online
textbook.
