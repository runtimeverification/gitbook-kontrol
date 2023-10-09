---
description: Cheatsheet with (almost) all of Kontrol options and gotchas
---

# Kontrol Cheatsheet

This cheatsheet provides a comprehensive list of options for conducting **Kontrol** verification of smart contracts, along with additional information about the **K** ecosystem.

## `kontrol` commands&#x20;

These commands encompass all the functionality involved in the verification process, from building your project into a full-fledged **K** definition to examining the verification output of a symbolic run in detail.

<table data-full-width="false"><thead><tr><th width="172.33333333333331">Command</th><th width="331">Description</th><th>Example</th></tr></thead><tbody><tr><td><code>build</code></td><td>Make <strong>K</strong> definitions from a Foundry project</td><td><code>kontrol build</code></td></tr><tr><td><code>prove</code></td><td>Symbolically execute the provided tests</td><td><code>kontrol prove --test TestContract.testName</code></td></tr><tr><td><code>list</code></td><td>List all proof files with their status</td><td><code>kontrol list</code></td></tr><tr><td><code>show</code></td><td>Statically print a proof</td><td><code>kontrol show ContractTest.testName</code></td></tr><tr><td><code>view-kcfg</code></td><td>Show a proof in the <strong>KCFG</strong> visualizer</td><td><code>kontrol view-kcfg ContractTest.testName</code></td></tr></tbody></table>

You can extensively customize the above commands to meet your specific verification requirements using the following flags.

***

## `kontrol build` &#x20;

### Flags

There are flags available for the `kontrol build` stage that allow you to change the set of lemmas used for reasoning and specify the desired behavior for rebuilding or re-executing symbolic tests.

<table data-full-width="false"><thead><tr><th>Flag</th><th>Description</th></tr></thead><tbody><tr><td><code>--verbose</code></td><td>Verbose build trace</td></tr><tr><td><code>--require $lemmas</code></td><td>Include a file of <code>$lemmas</code> when forming the <strong>K</strong> definition</td></tr><tr><td><code>--module-import $module</code></td><td><p>A <code>$module</code> from the <code>$lemmas</code> file provided in the above flag. </p><p><strong>Note:</strong> <code>$module</code> must be of the form <code>TestContract:ModuleName</code></p></td></tr><tr><td><code>--regen</code></td><td>Generate, even if it already exists, the file <code>foundry.k</code>. This file contains the project's <strong>K</strong> definition</td></tr><tr><td><code>--rekompile</code></td><td>Will rebuild the <strong>K</strong> definition, even if it was previously built</td></tr><tr><td><code>--with-llvm-library</code></td><td>Produce artifacts for the <a href="https://github.com/runtimeverification/llvm-backend">llvm backend</a>, needed to run the symbolic execution <a href="https://github.com/runtimeverification/hs-backend-booster">booster backend</a></td></tr></tbody></table>

### Chaining Flag Example&#x20;

Let's look at what a typical `kontrol build` example may look like. You will likely have the following contents:

| Content             | File Name            | File Path                  |
| ------------------- | -------------------- | -------------------------- |
| Custom Lemma        | `myproject-lemmas.k` | `test/myproject-lemmas.k`  |
| Module              | `MYPROJECT-LEMMAS`   | `test/myproject-lemmas.k`  |
| Symbolic properties | `MyProperties`       |                            |

* A _custom lemmas_
  * Lemmas file name: `myproject-lemmas.k`
  * This will be saved at following path: `test/myproject-lemmas.k`
* A _module_ you want **Kontrol** to include in its reasoning.&#x20;
  * Module name: `MYPROJECT-LEMMAS`
  * This will be saved at following path: `test/myproject-lemmas.k`&#x20;
* Symbolic properties to execute.
  * Contract file name: `MyProperties`&#x20;
  * This will be saved at following path:

For this example we will assume that you have run `kontrol build` once already and it is not the first time you are symbolically executing the properties. You've also now, identified necessary lemmas and included them in `MYPROJECT-LEMMAS`.&#x20;

You will need to `regen` and `rekompile` the **K** definition of the project again. You also want to use the fastest backend available (booster backend) to symbolically execute your properties. To do this your command will look like this:

```bash
kontrol build --require test/myproject-lemmas.k             \
              --module-import MyProperties:MYPROJECT-LEMMAS \
               --rekompile                                  \
               --regen                                      \
               --with-llvm-library
```

***

## `kontrol prove`&#x20;

### Flags

These flags specify what you `prove` and how you `prove`. The flags including the backend used for symbolic execution, the new lemmas to include for symbolic reasoning, resource distribute and other potential changes.

<table data-full-width="false"><thead><tr><th>Flag</th><th>Description</th></tr></thead><tbody><tr><td><code>--test $testName</code></td><td>Specifies the name of the test function to symbolically execute. Multiple flags can be provided for parallel execution with different functions.</td></tr><tr><td><code>--reinit</code></td><td>Restarts symbolic execution instead of resuming from the last saved run.</td></tr><tr><td><code>--bmc-depth $n</code></td><td>Enables bounded model checking, unrolling all loops to a maximum depth of <code>$n</code> .</td></tr><tr><td><code>--use-booster</code></td><td><p>Uses the booster backend (faster) instead of the legacy backend. </p><p><strong>Note:</strong> This option requires <a href="https://app.gitbook.com/o/MwuC1PgHx91Qm96rVCnq/s/T2KVb4tqbNdAsPxsEyPQ/~/changes/73/learn-more/kontrol-cheatsheet#kontrol-build-flags">building</a> the project with the <code>--with-llvm-library</code> flag</p></td></tr><tr><td><code>--smt-timeout $ms</code></td><td>Sets the timeout in milliseconds for SMT queries.</td></tr><tr><td><code>--smt-retry-limit $n</code></td><td>Specifies the number of times to retry SMT queries with scaling timeouts.</td></tr><tr><td><code>--auto-abstract-gas</code></td><td>Abstract any gas-related computations, provided the cheatcode <code>infiniteGas</code> was enabled. This simplifies and speeds up symbolic execution.</td></tr><tr><td><code>--no-break-on-calls</code></td><td>Does not store a node for every EVM call made.</td></tr><tr><td><code>--workers $n</code></td><td>Sets the number of parallel processes run by the prover. It should be at most <code>(M - 8) / 8</code> in a machine with <code>M</code> GB of RAM.</td></tr><tr><td><code>--max-depth $n</code></td><td>Sets the maximum number of <strong>K</strong> steps before the state is saved in a new node in the <strong>KCFG</strong>.</td></tr><tr><td><code>--max-iterations $n</code></td><td>Sets the number of times to expand the next pending node in the <strong>KCFG</strong>.</td></tr><tr><td><code>--bug-report $name</code></td><td>Generates a bug report with the given name.</td></tr></tbody></table>

### Example

Let's look at what a typical `kontrol prove` example may look like. You will likely have the following contents:

* Two symbolic properties to execute in parallel named `testMyProperty1` and `testMyProperty2`, both in a contract named `MyProperties`.
* There are loops present in the code, and before providing invariants for those we want to  use the simpler approach of bounded model checking, unrolling the loops only up to 10 iterations
* We want to allow the SMT solver enough time to reason
* We can achieve maximum speed of symbolic execution with the following tweaks
  * Don't allocate any resources to gas computations, since these are costly and can cause numerous branching
  * Create as few nodes as possible, since this saves on writing time. In particular, don't produce any nodes when making EVM calls
  * Use the fastest symbolic execution backend available, the booster backend

The command to execute `testMyProperty1` and `testMyProperty2`, with that set of characteristics is the following:

{% code overflow="wrap" fullWidth="false" %}
```bash
kontrol prove --test MyProperties.testMyProperty1 \
              --test MyProperties.testMyProperty2 \
              --bmc-depth 10                      \
              --smt-timeout 10000                   \
              --auto-abstract-gas                 \
              --no-break-on-calls                 \
              --workers 2                         \
              --use-booster # `kontrol build` must be run with --with-llvm-library
```
{% endcode %}

***

## Recommended Workflow

To make use of for all `kontrol` options, it is recommended you use a bash script that simplifies parameter tweaking and allows for a better verification experience. It is also recommended you save the output of running `kontrol` to a file for easier inspection and debugging.

Here is a template to have better control over `kontrol`. To save the output of running a template `run-kontrol.sh` to `log.out` , from the root directory of a Foundry project, you can use the following command:

```bash
time bash test/run-kontrol.sh 2>&1 | tee log.out
```

To interpret the result of running this script, please refer to [k-control-flow-graph-kcfg.md](../guides/kevm-foundry-integration-example/k-control-flow-graph-kcfg.md "mention").

### Script

```bash
#!/bin/bash

set -euxo pipefail

kontrol_build() {
    kontrol build                     \
            --verbose                 \
            --require ${lemmas}       \
            --module-import ${module} \
            ${rekompile}              \
            ${regen}                  \
            ${llvm_library}
}

kontrol_prove() {
    kontrol prove                              \
            --max-depth ${max_depth}           \
            --max-iterations ${max_iterations} \
            --smt-timeout ${smt_timeout}       \
            --bmc-depth ${bmc_depth}           \
            --workers ${workers}               \
            ${reinit}                          \
            ${bug_report}                      \
            ${break_on_calls}                  \
            ${auto_abstract}                   \
            ${tests}                           \
            ${use_booster}
}

###
# kontrol build options
###
lemmas=test/myproject-lemmas.k
base_module=MYPROJECT-LEMMAS
module=MyProperties:${base_module}

regen=--regen
#regen=

rekompile=--rekompile
#rekompile=

llvm_library=--with-llvm-library
# llvm_library=

###
# kontrol prove options
###
max_depth=10000

max_iterations=10000

smt_timeout=100000

bmc_depth=10

workers=2

reinit=--reinit
reinit=

break_on_calls=--no-break-on-calls
# break_on_calls=

auto_abstract=--auto-abstract-gas
# auto_abstract=

bug_report=--bug-report
#bug_report=

# If this option is enabled, --with-llvm-library must be too
use_booster=--use-booster
# use_booster=

# List of tests to symbolically execute
tests=""
tests+="--test MyProperties.testMyProperty1 "
tests+="--test MyProperties.testMyProperty2 "

pkill kore-rpc || true
kontrol_build
kontrol_prove

```
