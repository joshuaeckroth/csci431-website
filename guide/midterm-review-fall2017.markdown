---
layout: note
title: Midterm review
categories: [studyguides]
date: Wed Oct 11
---

# Midterm review

## Basic Prolog

Be able to identify if two forms "unify," and if there are any variables involved, what value those variables must have so the two forms unify.

Know how to write Prolog code that represents a given set of facts and rules, stated in English. Here are some examples:

- Familial relations: mother, father, aunt, uncle, niece, nephew, cousin, half-brother, half-sister, etc.
- Or, from Monty Python: "A witch is a female who burns. Witches burn - because they're made of wood. Wood floats. What else floats on water? A duck; if something has the same weight as a duck it must float." (quoted from [this Prolog tutorial](http://www.allisons.org/ll/Logic/Prolog/Examples/witch/)) The resulting Prolog code should be able to prove that witches float.

Be able to write recursive rules that perform various operations on lists, e.g., reversing a list or counting all the unique elements.

## Planning in Prolog

Given facts/rules about connections, e.g., a road network, be able to find a path from one point to another. E.g.,

```
road(a, b).
road(a, c).
road(b, d).
road(d, e).
road(c, e).
...etc...

% write rules to find a series of steps between two points, e.g. 'a' to 'e' gives [a,b,d,e].
```

## Parsing in Prolog

Be able to write a working grammar for a simple language, and to extract certain fields from the syntax (like we did in CiteMan). For simplicity, numbers will not be involved. But, the grammar may require recursive rules.



