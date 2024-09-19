---
layout: post
title: "CodeQL: Eindhoven Quantifier Notation"
categories: [security, tooling, sast]
tags: [programming, productivity]
description: "This blog post will discuss about Eindhoven Quantifier Notation adopted by CodeQL"
comments: true
---

### Introduction

Recently, I have been thinking about aggregate functionality design for [Code PathFinder](https://codepathfinder.dev/), [opensource alternative to GitHub CodeQL](https://github.com/shivasurya/code-pathfinder). SQL aggregate functions such as `SUM`, `AVG`, `MIN`, `MAX` are combined with `WHERE` and `GROUP BY` to generate aggregate queries. However, I was wondering if there is a way to generate aggregate queries without using `WHERE` and `GROUP BY`. While going through [CodeQL design research paper](https://codeql.github.com/publications/ql-for-source-code-analysis.pdf), I came across Eindhoven Quantifier Notation which is quite interesting, easy to understand and can be used to generate aggregate queries. This blog post will discuss about Eindhoven Quantifier Notation adopted by CodeQL.

### Eindhoven Quantifier Notation

Eindhoven Quantifier Notation (by Edsger Wybe Dijkstra) is useful in topics like logic and set theory. In traditional mathematical notation, quantifiers such as "for all" (universal quantifier) and "there exists" (existential quantifier) are used to specify the scope of variables within logical expressions. The Eindhoven notation modifies this by providing a structured format that makes the scope and constraints explicit.

The general form of an Eindhoven quantifier expression is: (Q x: T ∣ rng ⋅ exp)


Q: The quantifier, such as "∀" (for all), "∃" (there exists), or other operations like summation.

x: The variable being quantified.

T: The type or domain of the variable 

rng: A range condition that x must satisfy.

exp: The expression evaluated for each x

In simple terms, I would call as aggregate function (sum, max, min etc) combined with map and filter function in javascript. Moreoever it's simple to understand and write the expression. For example, in CodeQL, the following expression will return the total number of lines under package "abc.aspectj.ast":

```sql
import java

from Package p
where p.hasName("abc.aspectj.ast")
select sum(CompilationUnit cu | cu.getPackage()=p | cu.getNumberOfLines())
```

`sum(CompilationUnit cu | cu.getPackage()=p | cu.getNumberOfLines())` where `cu` is quantifier (for all) variable and in the next step gets filtered by package name and finally get the number of lines as Integer which gets added to the sum. This notiation gets executed for each compilation unit and compared with package name (global variable) before adding to the sum.

I really like the above query has basically filtered the package name first before generating the sum thus reducing the number of times the `cu.getPackage()=p` gets compared thus reducing the search space.


### Closing Note:

I really like the Eindhoven Quantifier Notation and it's simple to understand and write the expression. The curiosity of the mathematical notation got me to write this blog post. I hope you find this blog post useful. For bugs or hugs & discussion, DM in [Twitter](https://twitter.com/sshivasurya). Opinions are my own and not the views of my employer.