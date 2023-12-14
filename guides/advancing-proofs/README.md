---
description: How to identify and write lemmas to  advance on your proofs
---

# Advancing Proofs

{% hint style="info" %}
**Prerequisites:** Before proceeding with this section, it is recommended that you have already gone through the [kevm-foundry-integration-example](../kevm-foundry-integration-example/ "mention") and have a basic understanding of the [**K**](https://github.com/runtimeverification/k) framework and [**KEVM**](https://github.com/runtimeverification/evm-semantics). For more information, refer to [resources.md](../../learn-more/resources.md "mention") page.
{% endhint %}

***

## Manual Intervention

During the verification process using [**Kontrol**](https://github.com/runtimeverification/kontrol), there are situations where manual intervention is necessary to help the tool reason correctly and advance the symbolic execution. Two common situations involve `stuck` nodes and `invalid` execution paths.&#x20;

* `stuck` **nodes:** These nodes have not reached the end of execution, have not been subsumed into the target node, and **Kontrol** sees no further execution steps.
* `invalid` **execution paths:** It is not always easy for **Kontrol** to deduce that a particular execution path is infeasible. If an invalid execution path is traversed, it means that **Kontrol** failed to identify a contradiction from the path condition.

When these situations arise, the way to progress past them is to identify the missing reasoning links and make them available to **Kontrol** as Lemmas.

## Intro to Lemmas

You can think of lemmas as one-step reasoning processes that you want **Kontrol** to be aware of while symbolically executing your properties.

As rewrite rules, lemmas consist of a left-hand side (LHS) and a right-hand side (RHS). When **Kontrol** identifies a symbolic match for the LHS in the **K** configuration of a proof, it applies the rule, rewriting the LHS to the RHS.&#x20;

### Structure of a Lemma

The general structure of a lemma is as follows:

```
rule [name-of-lemma]: <k> LHS(X, Y) => RHS(X, Y) </k>
  requires condition1(X)
  andBool condition2(Y)
  [simplification]
```

In short, lemmas are simplification rules, specifically, **K** rewrite rules with the `simplification` attribute. Other attributes can be added alongside `simplification`. Here is a list of commonly used attributes when defining lemmas:

* `smt-lemma`: passes the simplification rule down to the SMT-solver. To use this predicate, the functions appearing in the lemma must be defined with the `smt-lib` attribute
* `concrete(VAR)`: applies the rule only when `VAR` can be matched to a concrete value
* `symbolic(VAR)`: applies the rule only when `VAR` can be matched to a symbolic value
* `comm`: TBD

Multiple attributes can be separated by commas, for example: `[simplification, concrete(X), symbolic(Y)]`. Additionally, instead of the `andBool` connective, `orBool` can also be used.

### Lemmas and Your Verification Project

To add lemmas to your project, have a file `myproject-lemmas.k`, usually under the `kontrol/` directory, with the following structure:

```
requires "evm.md"
requires "foundry.md"

module MYPROJECT-LEMMAS
    imports BOOL
    imports FOUNDRY
    imports INFINITE-GAS
    imports INT-SYMBOLIC

// Your lemmas go here

endmodule
```

Along with creating this file, **Kontrol** should be made aware of the lemmas contained in the file to reason with. This can be done in the building phase of your project. To include `test/myproject-lemmas.k` into the reasoning capabilities of Kontrol, the following flags should be included:

* `--require test/myproject-lemmas.k`
* `--module-import MyPojectTests:MYPROJECT-LEMMAS`
* `--rekompile` (optional)

The `--require` flag tells from which file the module `MYPROJECT-LEMMAS` is imported, specified in the `--module-import` flag. The `MyProjectTests:` string which precedes the module name is the name of the contract of the tests to be symbolically executed. Finally, `--rekompile` will rebuild the project. It is necessary to include the `--rekompile` flag when adding new lemmas, otherwise **Kontrol** won't be aware of them. Thus, when new lemmas are added, the project should be built as follows, from the root directory:

{% code fullWidth="true" %}
```
kontrol build --require kontrol/myproject-lemmas.k --module-import MyPojectTests:MYPROJECT-LEMMAS
```
{% endcode %}

### Finding Lemmas

In the following sections, we'll explore how to find two different kinds of lemmas, **KEVM** and arithmetical lemmas. **KEVM** lemmas are lemmas that fill reasoning gaps at the EVM semantics level, while arithmetical lemmas fill, well, arithmetical gaps.
