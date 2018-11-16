---
title: Rules
keywords: graql, automated reasoing, machine reasoning
tags: [graql, reasoning]
summary: "How Grakn uses rules to reason over explicitly stored data."
permalink: /docs/schema/rules
---

## Introduction {#introduction1111}
Grakn is capable of reasoning over data via pre-defined rules. Graql rules dyanmically create relationships that were non-existant when the raw data was initially inserted into the knowledge graph. [Automated reasoning](...) provided by rules is performed at query time and is guaranteed to be complete. Rules not only allow shortenening and simplifying commonly-used queries but also enable implementation of business logic at the database level.

When you query the knowledge graph for certain information, Grakn returns a complete set of answers. These answeres contain direct matches as well as those inferred by the rules included in the schema.

In this section, we will learn more about how rules are constructed and how they are meant to be used.

## Defining a Rule
Defining a Graql rule begins with a given name followed by `sub rule`, the `when` body as the condition, and the `then` body as the conclusion.

```graql
rule-label sub rule,
  when {
    ## the condition
  } then {
    ## the conclusion
  };
```

Let's look an example.

```graql
define

people-with-same-parents-are-siblings sub rule,
  when {
    (mother: $m, $x) isa parentship;
    (mother: $m, $y) isa parentship;
    (father: $f, $x) isa parentship;
    (father: $f, $y) isa parentship;
    $x != $y;
  } then {
    ($x, $y) isa siblings;
  };
```

The Graql rule above is telling Grakn the following:

when
: there are two different people (`$x` and `$y`) who have the same mother `$m` and the same father `$f`,

then
: those people (`$x` and `$y`) must be siblings.

If the find the Graql code above unfamiliar, don't be concerned. We will soon learn about [using Graql to describe patterns](...).

In this example, siblings data is not explicitly stored anywhere in the knowledge graph. But by having included this rule in the schema, we can always know who the siblings are and and include the `siblings` relationship in our queries.

This is a basic example of how Graql rules can be useful. In a dedicated section, we learn about rules by looking at more examples of [rule-based automated reasoning](...).

## The Underlying Logic
Under the hood, rules are restricted to be definite Horn Clauses. In simple terms, the Graql statements placed in the `when` body form one single condition where all statements must be true for the rule to apply. The `then` body on the other hand is restricted to contain one single statement.

To simplify this logic even further, you can think of the [siblings example](#defining-a-rule) in form of an `if` statement like so:

```
if (is-m-mother-of-x && is-m-mother-of-y && is-f-father-of-x && is-f-father-of-y) {
    are-x-and-y-siblings = true;
    // any more assignments will break the rule!
}
```
{% include warning.html content = 'The text below down to the next subtitle will be put in a panel labeled with "Advnanced Topic"' %}
Insight: Rules as Horn Clauses can be defined either in terms of a disjunction with at most one unnegated atom or an implication with the consequent consisting of a single atom. Atoms are considered atomic first-order predicates - ones that cannot be decomposed to simpler constructs.
In our system we define both the head and the body of rules as Graql patterns. Consequently, the rules are statements of the form:

```
q1 ∧ q2 ∧ ... ∧ qn → p
```

where qs and the p are atoms that each correspond to a single Graql statement. The “when” of the statement (antecedent) then corresponds to the rule body with the “then” (consequent) corresponding to the rule head.

The implication form of Horn clauses aligns more naturally with Graql semantics as we define the rules in terms of the “when” and “then” blocks which directly correspond to the antecedent and consequent of the implication respectively.

### What can go in the then body
The following are the types of a single statement that can be set as the conclusion of a rule in the `then` body:
- setting the type. Example: `$x isa person;`
- assigning an explicit value to an attribute. Example: `$x has age 20;`
- inserting a relationship. Example: `(parent: $x, child: $y) isa parentship;`

## Deleting Rules
Rules like any other schema element can be undefined. To do so, you may learn [how to use the undefine keyword](/docs/schema/concepts#undefine) for deleting rules.

## Summary