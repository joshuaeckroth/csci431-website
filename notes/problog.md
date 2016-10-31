---
title: ProbLog
layout: note
---

# ProbLog

ProbLog is a redesign and new implementation of [Prolog](/notes/prolog.html) in which facts and rules can be annotated with probabilities. Have a look at their [website](https://dtai.cs.kuleuven.be/problog/index.html), especially their tutorials.

> ProbLog is a Python package and can be embedded in Python or Java. Its knowledge base can be represented as Prolog/Datalog facts, CSV-files, SQLite database tables, through functions implemented in the host environment or combinations hereof.

## Syntax

Facts and rules can be annotated with probabilities by adding a floating-point number in front of the fact/rule, followed by double-colons. For example (from ProbLog's site):

```prolog
0.3::stress(X) :- person(X).
0.2::influences(X,Y) :- person(X), person(Y).

smokes(X) :- stress(X).
smokes(X) :- friend(X,Y), influences(Y,X), smokes(Y).

0.4::asthma(X) :- smokes(X).

person(angelika).
person(joris).
person(jonas).
person(dimitar).

friend(joris,jonas).
friend(joris,angelika).
friend(joris,dimitar).
friend(angelika,jonas).
```

You would also typically add some evidence and a query to the end of the program, e.g.,

```
evidence(asthma(jonas)).
query(smokes(angelika)).
```

On londo, you can run a ProbLog program like so:

```
$ problog smokes.pl
smokes(angelika):       0.44
```

Each `query()` predicate produces an output. Use a variable in the query to iterate over the possibilities:

```prolog
query(smokes(X)).
```

Result:

```
$ problog smokes.pl
smokes(angelika):       0.44
 smokes(dimitar):       0.3
   smokes(jonas):       1
   smokes(joris):       0.5199232
```

## Example: Toothache

Consider the toothache example from the [Bayesian inference](/notes/bayesian-inference.html) notes. We can solve it with ProbLog as follows:

```prolog
0.10::catch.
0.05::gum_disease.

1.00::toothache :- catch, gum_disease.
0.60::toothache :- catch, \+gum_disease.
0.30::toothache :- \+catch, gum_disease.
0.05::toothache :- \+catch, \+gum_disease.

query(toothache).
```

Run the program on the command line:

```
$ problog toothache.pl
toothache:      0.11825
```

Now suppose we observe toothache. What's the probability of "catch"? ProbLog will perform Bayesian inference to satisfy that query. First, update the last line of the program:

```prolog
evidence(toothache, true).
query(catch).
```

And run it:

```
problog toothache.pl
catch:  0.5243129
```

## Example: Report of a fire

```prolog
0.02::tampering.
0.01::fire.

0.50::alarm :- tampering, fire.
0.85::alarm :- tampering, \+fire.
0.99::alarm :- \+tampering, fire.
0.00::alarm :- \+tampering, \+fire. % or we could just leave this out, since it's 0.0% anyway

0.90::smoke :- fire.
0.01::smoke :- \+fire.

0.88::leaving :- alarm.
0.00::leaving :- \+alarm. % or we could just leave this out, since it's 0.0% anyway

0.75::report :- leaving.
0.01::report :- \+leaving.


% first, forward inference (no Bayesian inference)
evidence(tampering, false).
evidence(fire, true).
query(report).

% second, backwards inference (Bayesian inference)
%evidence(report, true).
%query(fire).
```

## Example: Who stole the cookies?

- A plate is empty.
- Marsha says the plate was stacked with cookies last night. When pressed further, she claims she's "99% sure of this."
- The plate is on a counter. The family dog, Max, has been seen to eat food off the counter. It's possible, say 10% chance, that the dog could have reached the plate of cookies.
- The cookies had chocolate chips. Chocolate is poisonous to dogs. The [Dog Chocolate Toxicity Meter](http://www.petmd.com/dog/chocolate-toxicity) tells us that Max, who weighs 50lbs, would experience GI upset, vomiting, and shaking after eating 6oz of milk chocolate (the amount we estimate was in the cookies). The dog is showing no symptoms, or he hides it well.
- Marsha says she hates chocolate chip cookies, in spite of how implausible that sounds.
- Tom made the cookies. Tom loves cookies. Tom usually eats all the cookies he makes (80% of the time). Sometimes he does it when no one is looking (40% of the time). But Tom says he didn't eat the cookies. The family believes Tom's lies about 20% of the time.

Who ate the cookies?

```prolog
% A plate is empty.
%
evidence(plate_with_cookies(now), false).

% Marsha says the plate was stacked with cookies last night. When pressed
% further, she claims she's "99% sure of this."

0.99::plate_with_cookies(last_night).

% The plate is on a counter. The family dog, Max, has been seen to eat food off
% the counter. It's possible, say 10% chance, that the dog could have reached
% the plate of cookies.

0.10::dog_reach_plate.

% The cookies had chocolate chips. Chocolate is poisonous to dogs. The Dog
% Chocolate Toxicity Meter tells us that Max, who weighs 50lbs, would
% experience GI upset, vomiting, and shaking after eating 6oz of milk chocolate
% (the amount we estimate was in the cookies). The dog is showing no symptoms,
% or he hides it well.

0.80::believe(dog).

% Marsha says she hates chocolate chip cookies, in spite of how implausible
% that sounds.

% (just don't write likes_cookies for marsha).

% Tom made the cookies. Tom loves cookies. Tom usually eats all the cookies he
% makes (80% of the time). Sometimes he does it when no one is looking (40% of
% the time). But Tom says he didn't eat the cookies. The family believes Tom's
% lies about 20% of the time.

likes_cookies(tom).

0.20::believe(tom).

% Note: we have no reason not to believe Marsha (according to the story anyway).

1.00::believe(marsha).

% Background logic.

ate_cookies(dog) :-
    plate_with_cookies(last_night),
    \+plate_with_cookies(now),
    believe(dog),
    dog_reach_plate.

ate_cookies(X) :-
    plate_with_cookies(last_night),
    \+plate_with_cookies(now),
    believe(X),
    likes_cookies(X).

% Who ate the cookies?

query(ate_cookies(X)).
```

Run the program:

```
$ problog cookies.pl
ate_cookies(dog):       0.0792
ate_cookies(tom):       0.198
```

Looks like the dog's off the hook.