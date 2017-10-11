---
layout: note
title: Midterm for Fall 2017
categories: [tests]
---

# Midterm for Fall 2017

Submit your solutions on bitbucket in a repository named csci431-midterm. **Answers are due by Fri Oct 13, 11:59pm.** You may look at our course notes, assignments, code I wrote in my [`teach-prolog` GitHub repo](https://github.com/joshuaeckroth/teach-prolog), and the [SWI Prolog documentation](http://www.swi-prolog.org/).

## Question 1 (5pts)

Create a Prolog program that counts and prints votes. A vote is just a string (e.g., a person's name). The votes are assumed to be in a list. The `countAndPrintVotes(Votes)` predicate should count and print the votes in sorted order: most votes first. If two people have the same number of votes, show in any order. Create additional predicates as needed to complete the task. For sorting, consider using the [sort/4](http://www.swi-prolog.org/pldoc/man?predicate=sort/4) predicate. Use the [`format/2`](http://www.swi-prolog.org/pldoc/doc_for?object=format/2) predicate to print each line.

Example usage:

```
?- countAndPrintVotes(["Josh", "Bob", "Amy", "Amy", "Josh", "Josh", "Bob", "Amy", "Josh"]).
Josh: 4
Amy: 3
Bob: 2
true.
```

FYI, my solution had 18 lines of code and four predicates (sometimes multiple versions of each predicate).

## Question 2 (5pts)

Write a parser that is equivalent to the regex below, and extracts (returns) whatever matched the parts in parentheses:

```
\w+\s*(foo|bar)s?q(..)x
```

If it helps, feel free to look at the regex grounding code I showed in class: [`regexgrounder.pl`](https://github.com/joshuaeckroth/teach-prolog/blob/master/regexgrounder.pl) (and any of the other code in that repo). For the `\w` character matcher, use `char_type(X, csym)` (see [documentation](http://www.swi-prolog.org/pldoc/doc_for?object=char_type/2)). Be sure to start your code with `:- set_prolog_flag(double_quotes, chars).`

Example usage:

```
?- regex(Group1, Group2, "abc foosqQWx", []).
Group1 = [f, o, o],
Group2 = ['Q', 'W'] ;
false.

?- regex(Group1, Group2, "abcbarsq12x", []).
Group1 = [b, a, r],
Group2 = ['1', '2'] ;
false.

?- regex(Group1, Group2, "abcbarsqx", []).
false.

?- regex(Group1, Group2, "xbarsq**x", []).
Group1 = [b, a, r],
Group2 = [*, *] ;
false.

?- regex(Group1, Group2, "___    foosq**x", []).
Group1 = [f, o, o],
Group2 = [*, *] ;
false.
```

FYI, my solution had 10 lines of code and six predicates (sometimes multiple versions of each predicate).

## Question 3 (5pts)

Finish the code below to build a unit conversion program. Using the CLPR (constraint logic programming with reals) we can write predicates that describe how to convert one unit to another, and the CLPR library can automatically convert backwards as well. Thus, using biductive programming, we can run the `convert_u` predicates with any/all arguments as variables or literals. Your task is to write the predicates that find a chain of conversions if one exists, e.g., from lb to kg or kg to lb or days to seconds, etc. Obviously, you should support the base case, where the units are the same, e.g., kg to kg.

```
:- use_module(library(clpr)).

% supported units (to prevent infinite recursion)
unit(kg).
unit(g).
unit(lb).
unit(minutes).
unit(seconds).
unit(hours).
unit(days).

convert_u((X, U), (X, U)).

% weights

convert_u((X, kg), (Y, lb)) :-
    { X = 0.453592 * Y }.

convert_u((X, kg), (Y, g)) :-
    { X = 0.001 * Y }.

% time

convert_u((X, seconds), (Y, minutes)) :-
    { X = 60 * Y }.
convert_u((X, minutes), (Y, hours)) :-
    { X = 60 * Y }.
convert_u((X, hours), (Y, days)) :-
    { X = 24 * Y }.

% TODO: define convert((X, U1), (Y, U2)) predicate(s).
```

Test cases:

```
% run as: swipl -s tests.pl -t run_tests

:- use_module(library(test_cover)).

:- begin_tests(units).

:- [units].

% compare floats
fequal(X, Y) :-
    Z is abs(X-Y),
    Z < 0.001.

test(weights) :- convert((3, kg), (Y, kg)), fequal(3, Y).
test(weights) :- convert((3, kg), (Y, lb)), fequal(6.6139, Y).
test(weights) :- convert((X, lb), (3, kg)), fequal(6.6139, X).
test(weights) :- convert((3, kg), (Y, g)), fequal(3000, Y).
test(weights) :- convert((X, g), (3, kg)), fequal(3000, X).
test(weights) :- convert((0.75, lb), (Y, g)), fequal(340.194, Y).
test(weights) :- convert((X, g), (0.75, lb)), fequal(340.194, X).

test(time) :- convert((5, minutes), (Y, seconds)), fequal(300, Y).
test(time) :- convert((X, minutes), (300, seconds)), fequal(5, X).
test(time) :- convert((5, days), (Y, seconds)), fequal(432000, Y).
test(time) :- convert((X, days), (432000, seconds)), fequal(5, X).

test(invalid_conversions, [fail]) :- convert((5, days), (_, lb)).
test(invalid_conversions, [fail]) :- convert((_, days), (5, lb)).
test(invalid_conversions, [fail]) :- convert((5, kg), (_, seconds)).
test(invalid_conversions, [fail]) :- convert((_, kg), (5, seconds)).
test(invalid_conversions, [fail]) :- convert((_, foo), (_, seconds)).
test(invalid_conversions, [fail]) :- convert((_, kg), (_, foo)).

:- end_tests(units).
```

FYI, my solution had 24 lines of code and two predicates (multiple versions of one of those predicates, but in this case the variants were all very similar).


