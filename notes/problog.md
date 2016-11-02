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

You would also typically add a query to the end of the program, e.g.,

```
query(smokes(angelika)).
```

On londo, you can run a ProbLog program like so:

```
$ problog smokes.pl
smokes(angelika):       0.342
```

Each `query()` predicate produces an output. Use a variable in the query to iterate over the possibilities:

```prolog
query(smokes(X)).
```

Result:

```
$ problog smokes.pl
smokes(angelika):       0.342
 smokes(dimitar):       0.3
   smokes(jonas):       0.3
   smokes(joris):       0.42301296
```

We can also add "evidence" which restricts the search for proofs:

```prolog
% add to smokes.pl:
evidence(smokes(jonas)).
```

Now the result is:

```
$ problog smokes.pl
smokes(angelika):       0.44
 smokes(dimitar):       0.3
   smokes(jonas):       1
   smokes(joris):       0.5199232
```

We can also force evidence to be false:

```prolog
evidence(smokes(jonas), false).
```

Thus,

```
$ problog smokes.pl
smokes(angelika):       0.3
 smokes(dimitar):       0.3
   smokes(joris):       0.38148
```

## Probabilistic semantics

Given the syntax example above, we have enough knowledge to create probabilistic programs. However, it is important to know what it *means* to label a fact/rule with a probability. Note, an understanding of [Prolog](/notes/prolog.html) is important to understanding ProbLog's computational model.

ProbLog makes the assumption that all probabilistic facts are *mutually independent*. Recall that if we have independent random variables $a$ and $b$, then $P(a,b)=P(a)P(b)$. This simple multiplication can use the probabilities for $a$ and $b$ specified by the user in the ProbLog program. If they are not assumed to be independent, we have the more tricky $P(a,b)=P(a|b)P(b)$, which involves a probability $P(a|b)$ which, presumably, the user has not specified in the program.

The probability of a ProbLog query `query(f)` given a ProbLog program (facts and rules) `T`, which we can write as $P(q|T)$, is defined as:

$$P(q|T)=\sum_{L} P(q|L)P(L|T),$$

where $L$ ranges over facts/predicates, $P(q|L)=0$ if $q$ is not true given $L$ or $P(q|L)=1$ if $q$ is true given $L$, and $P(L|T)$ is defined as:

$$P(L|T)=\prod_{c_i \in L} p_i \prod_{c_j \in L'} (1-p_j),$$

where $c_i \in L$ ranges among the facts mentioned in $L$, and $c_j \in L'$ ranges among all the facts not mentioned in $L$.

In other words, the probability of a particular set of facts $L$ for a given program $T$, written $P(L|T)$, is the multiplication (product) of the probability of each of those facts being true (as specified in the ProbLog program) times the product of the probability of all the unmentioned facts being false. This simple multiplication is possible because of the assumption (by ProbLog) that every fact is mutually independent, so their probabilities can just be multiplied.

In summary, the probability of a particular query $q$ being true in a program $T$, written $P(q|T)$, is equal to the sum of probabilities of the different ways (proofs) the query is true. This is equivalent to *marginalization*, discussed on the [Bayesian inference](/notes/bayesian-inference.html#tocAnchor-1-3) notes.

In the smoking example above, first imagine there are no probabilities, so we have a normal Prolog program:

```prolog
stress(X) :- person(X).
influences(X,Y) :- person(X), person(Y).

smokes(X) :- stress(X).
smokes(X) :- friend(X,Y), influences(Y,X), smokes(Y).

asthma(X) :- smokes(X).

person(angelika).
person(joris).
person(jonas).
person(dimitar).

friend(joris,jonas).
friend(joris,angelika).
friend(joris,dimitar).
friend(angelika,jonas).

% "evidence": state that jonas has asthma
asthma(jonas).
```

The query `smokes(angelika)` has the following proofs (and only these):

- `person(angelika)` proves `stress(angelika)`, which proves `smokes(angelika)`.
  - Facts involved: `person(angelika)`, `stress(angelika)`.
- `friend(angelika, jonas)` with `influences(jonas, angelika)` (which is proved by `person(jonas)` and `person(angelika)`) with `smokes(jonas)` (which is proved by `person(jonas)` and `stress(jonas)`) altogether proves `smokes(angelika)`.
  - Facts involved: `person(angelika)`, `person(jonas)`, `friend(angelika, jonas)`, `influences(jonas, angelika)`, `stress(jonas)`, `smokes(jonas)`.

Looking back at the original ProbLog version of this program (shown at the top of the Syntax section on this page), we have the following probabilities involved in the two proofs of `smokes(angelika)`:

- 0.3: `stress(angelika)`
- 0.7: not `stress(angelika)` (needed to skip past first case); 0.2: `influences(jonas, angelika)`; 0.3: `stress(jonas)`.

The resulting probability is, therefore, $0.3+0.7*0.2*0.3 = 0.342$.

If we add `evidence(smokes(jonas))`, then the second proof does not need to involve `stress(jonas)`, thus dropping the $0.3$ multiplier, resulting in: $0.3+0.7*0.2=0.44$.

If instead we add `evidence(smokes(jonas), false)`, then the second proof completely fails, leaving only the first proof with probability $0.3$.

## Example: Toothache

Consider the toothache example from the [Bayesian inference](/notes/bayesian-inference.html) notes. We can solve it with ProbLog as follows:

```prolog
0.10::cavity.
0.05::gum_disease.

1.00::toothache :- cavity, gum_disease.
0.60::toothache :- cavity, \+gum_disease.
0.30::toothache :- \+cavity, gum_disease.
0.05::toothache :- \+cavity, \+gum_disease.

query(toothache).
```

Run the program on the command line:

```
$ problog toothache.pl
toothache:      0.11825
```

Now suppose we observe toothache. What's the probability of "cavity"? ProbLog will perform Bayesian inference to satisfy that query. First, update the last line of the program:

```prolog
evidence(toothache).
query(cavity).
```

And run it:

```
problog toothache.pl
cavity:  0.5243129
```

Or instead, we could indicate we have evidence there is no toothache:

```
evidence(toothache, false).
```

Now we get:

```
$ problog toothache.pl
cavity: 0.043096116
```

Perhaps most interesting, if we had to decide one cause (cavity or gum disease), which is most likely, given a toothache?

```prolog
evidence(toothache).
query(cavity).
query(gum_disease).
```

Result:

```
$ problog toothache.pl
     cavity:    0.5243129
gum_disease:    0.1564482
```

Thus, we find out a cavity is much more likely given a toothache.

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
% evidence(report, true).
% query(fire).
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