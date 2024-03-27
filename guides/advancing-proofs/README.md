---
description: How to identify and write lemmas to advance on your proofs
---

# Advancing Proofs

{% hint style="info" %}
**Prerequisites:** Before proceeding with this section, it is recommended that you have already gone through the [kontrol-example](../kontrol-example/ "mention") and have a basic understanding of the [**K**](https://github.com/runtimeverification/k) framework and [**KEVM**](https://github.com/runtimeverification/evm-semantics). For more information, refer to [resources.md](../../learn-more/resources.md "mention") page.
{% endhint %}

***

## Manual Intervention

During the verification process using [**Kontrol**](https://github.com/runtimeverification/kontrol), there are situations where manual intervention is necessary to help the tool reason correctly and advance the symbolic execution. Two common situations involve `stuck` nodes and invalid execution paths.&#x20;

* `stuck` **nodes:** These nodes have not reached the end of execution, have not been subsumed into the target node, _and_ **Kontrol** does not see any further execution steps.
* **Invalid execution paths:** It is not always easy for **Kontrol** to deduce that a particular execution path is infeasible. If an invalid execution path is traversed, it means that **Kontrol** failed to identify a contradiction from the path condition.

When these situations arise, the way to progress past them is to identify the missing reasoning links and make them available to **Kontrol** as Lemmas.

## Intro to Lemmas

Lemmas can be seen as one-step reasoning processes that you want **Kontrol** to consider during symbolic execution of your properties.

Similar to `rewrite` rules, lemmas have a left-hand side (LHS) and a right-hand side (RHS). When **Kontrol** identifies a symbolic match for the LHS in the **K** configuration of a proof, it applies the rule and rewrites the LHS to the RHS.&#x20;

### Structure of a Lemma

The general structure of a lemma is as follows:

```
rule [name-of-lemma]: <k> LHS(X, Y) => RHS(X, Y) </k>
  requires condition1(X)
  andBool condition2(Y)
  [simplification]
```

In short, lemmas are simplification rules, specifically, **K** rewrite rules with the `simplification` attribute. You can include other attributes in addition to `simplification`. Below is a list of commonly used attributes when defining lemmas:

* `smt-lemma`: passes the `simplification` rule down to the SMT-solver. To use this predicate, the functions in the lemma must be defined with the `smt-lib` attribute.
* `concrete(VAR)`: applies the rule only when `VAR` can be matched to a concrete value
* `symbolic(VAR)`: applies the rule only when `VAR` can be matched to a symbolic value

Multiple attributes can be separated by commas, for example: `[simplification, concrete(X), symbolic(Y)]`. Additionally, instead of the `andBool` connective, `orBool` can also be used.

### Lemmas and Your Verification Project

To add lemmas to your project, create a file called: `myproject-lemmas.k`. This file is usually located in the `kontrol/` directory and should have the following structure:

```
requires "foundry.md"

module MYPROJECT-LEMMAS
    imports BOOL
    imports FOUNDRY
    imports INFINITE-GAS
    imports INT-SYMBOLIC

// Your lemmas go here

endmodule
```

After creating this file, you need to inform **Kontrol** about the lemmas in the file so that it can reason with them. This can be done during the `build` phase of your project. To include `test/myproject-lemmas.k` in the reasoning capabilities of **Kontrol**, the following flags should be included:

* `--require test/myproject-lemmas.k` : specifies from which file the module `MYPROJECT-LEMMAS` is imported.
* `--module-import MyProjectTests:MYPROJECT-LEMMAS`: specifies the module to import. The module name is preceded by the string `MyProjectTests:` , which represents the contract name for the tests being symbolically executed.
* `--rekompile` (optional): rebuild the project. It is necessary to include the `--rekompile` flag when adding new lemmas, otherwise **Kontrol** won't be aware of them.&#x20;

You can find more information about `build` flags here: [#kontrol-build](../../cheatsheets/kontrol-cheatsheet.md#kontrol-build "mention")

To `build` with the flags above, run the following command:

{% code fullWidth="true" %}
```
kontrol build --require kontrol/myproject-lemmas.k --module-import MyProjectTests:MYPROJECT-LEMMAS
```
{% endcode %}

### Finding Lemmas

In the next sections, we will explore how to find  **KEVM** lemmas, which are used to address reasoning gaps at the EVM semantic level.
