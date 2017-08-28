---
title: Parsing with Prolog
layout: note
---

# Parsing with Prolog

(Note, SWI-Prolog must be run with the `--traditional` flag to work with the examples below.)

One of the original design goals of Prolog was to support natural language processing, including string parsing. Prolog supports a convenient syntax known as “definite clause grammars” (DCG). A set of DCG rules look like Backus-Naur Form. They are translated into predicates that break apart the input string into a list of characters, and each predicate resulting from DCG translation consumes a portion of the list of characters and leaves behind a remainder that is matched by another rule (or itself recursively). For example, a simple English sentence parser may be specified as follows.

    sentence --> nounPhrase, " ", verbPhrase, ".".
    nounPhrase --> determiner, " ", noun.
    verbPhrase --> verb, " ", nounPhrase.
    verbPhrase --> verb.
    determiner --> "the".
    determiner --> "a".
    noun --> "cat".
    noun --> "dog".
    verb --> "chases".
    verb --> "avoids".

Since parsing with DCGs in Prolog is equivalent to finding the appropriate splits of a list of character codes, we can use Prolog’s biductive computing capabilities to give partial arguments and let Prolog fill in the rest. For example, parsing a sentence like “a cat avoids the dog” returns “true,” i.e., the sentence predicate matches, but we can also generate different endings to the sentence by leaving the end of the sentence variable (indicated by `Rest`):

    append("a cat avoids", Rest, S), sentence(S, []).

The query yields possible completions " the cat.", " the dog.", " a cat.", and so on. Note that we add `[]` as a second argument to the sentence predicate to indicate we want no leftover characters after parsing.

DCGs may also be written in a way that allows them to accumulate "state" while parsing. For example, we can write a grammar consisting of just letters "a" and "b" and counts number of a's and b's:

    s(CountA, CountB) --> "a", s(PriorCountA, CountB), { CountA is PriorCountA + 1 }.
    s(CountA, CountB) --> "b", s(CountA, PriorCountB), { CountB is PriorCountB + 1 }.
    s(0, 0) --> [].

The arguments `CountA` and `CountB` precede the normal arguments for parsing when building a query:

    s(CountA, CountB, "aa", []).

The result is:

    CountA = 2,
    CountB = 0 .

Another try:

    s(CountA, CountB, "aabba", []).

Result:

    CountA = 3,
    CountB = 2 .

Notice the `{ CountA is PriorCountA + 1 }` part of the grammar is normal Prolog code. The `{}` constraints can appear anywhere in the grammar depending on when a variable needs to be constrained or computed. In this situation, `CountA` cannot be computed from `PriorCountA` until after `s(PriorCountA, CountB)` has finished because otherwise we do not yet know the value of `PriorCountA`. This applies to `PriorCountB` as well. Thus, the constraints appear after the recursive call.

We can define another version of the predicate `s` with just one extra argument (rather than two) that requires the counts of a's and b's to be the same:

    s(CountAB) --> s(CountAB, CountAB).

And run a query like so:

    s(CountAB, "aabba", []).

Result:

    false.

Another try:

    s(CountAB, "aabbab", []).

Result:

    CountAB = 3 .

Finally, if we try to use the grammar biductively by specifying the count but not the string, we get infinite recursion:

    s2(3, String, []). % oops, infinite recursion!

This is because the count variables are not properly constrained. We need to use the constraint logic programming for finite domains library (clpfd) to set bounds on the counts. Furthermore, we need the constraints to appear at the beginning of the grammar rules instead of the end so the constraints are established before we get into a recursive deep dive:

    :- use_module(library(clpfd)).

    s2(CountAB) -->
        { CountAB in 0..20 },
        s2(CountAB, CountAB).
    s2(CountA, CountB) -->
        { PriorCountA in 0..10, CountA #= PriorCountA + 1 },
        "a", s2(PriorCountA, CountB).
    s2(CountA, CountB) -->
        { PriorCountB in 0..10, CountB #= PriorCountB + 1 },
        "b", s2(CountA, PriorCountB).
    s2(0, 0) --> [].

First, we’ll check if the normal parsing (deductive parsing) works:

    s2(CountAB, "aabbab", []).

First result:

    CountAB = 3

Second result:

    false.

So far so good. Now let’s try an abductive (reversed) query, i.e., we give the count, and Prolog generates a string:

    s2(3, String, []), format("~s~n", [String]).

First result:

    aaabbb

Second result:

    aababb

And so on. Let’s count how many results we have:

    setof(String, s2(3, String, []), AllStrings), length(AllStrings, L).

Apparently there are 20 such strings with three a’s and three b’s. Mathematically, if we restrict the number of a’s and b’s to “n”, then we have (2n)!/(n!n!) variations.

In summary, we defined a grammar that accumulates state (the count of a's and b's) and also enforces a constraint on that state. The parser may be used in forwards and backwards directions, i.e., counting a's and b's from a string, or producing a string that has a given count of a's and b's. For what it's worth, the same code can also be asked to produce all strings and all counts that work (i.e., giving no information upfront).

The grammar specifying equal a's and b's is context-free. We can update the code to parse a non-context-free grammar such as requiring a series of a's, followed by a series of b's, followed by the same number of a's as the first series, and furthermore requiring there are more b's than a's:

    s3(CountA, CountB) --> s3a(CountA, CountB).
    s3a(CountA, CountB) -->
        { PriorCountA in 0..10, CountA #= PriorCountA + 1, CountB in 0..10, CountB #>= CountA },
        "a", s3a(PriorCountA, CountB), "a".
    s3a(0, CountB) --> s3b(CountB).
    s3a(0, 0) --> [].
    s3b(CountB) -->
        { PriorCountB in 0..10, CountB #= PriorCountB + 1 },
        "b", s3b(PriorCountB).
    s3b(0) --> [].

The above code supports these queries:

    s3(CA, CB, "aabbbbbbaa", []).
    ==> Result: CA = 2, CB = 6

    s3(3, 5, String, []), format("~s~n", [String]).
    ==> Result: aaabbbbbaaa

    setof((CA, CB, String), s3(CA, CB, String, []), Result), length(Result, L).
    ==> Result: L=67

The last query shows there are 67 strings that statisfy the grammar. Of course, there would be infinitely-many if we remove the limits of `CountA in 0..10` and `CountB in 0..10.`
