---
description: Investigating a failed test and understanding the KCFG output
---

# K Control Flow Graph (KCFG)

We need to investigate and identify the reason for the failed symbolic test. To do this, we can use the following command:

```bash
kontrol view-kcfg 'CounterTest.testSetNumber(uint256,bool)' --version 1
```

{% hint style="info" %}
Note:  the tag `--version 1` indicates that you want to view the **KCFG** for version 1 of the proof `CounterTest.testSetNumber(uint256,bool)`

If you would like to view the first proof you will run:

```
kontrol view-kcfg 'CounterTest.testSetNumber(uint256,bool)' --version 0
```

For more information check out the page on [proof-management.md](proof-management.md "mention").
{% endhint %}

This command launches an interactive visualizer that generates a **KCFG** (**K Control Flow Graph**). You can click on individual nodes in the **KCFG** to inspect them.

<figure><img src="../../.gitbook/assets/KCFG Vis.png" alt=""><figcaption><p>KCFG Visualizer</p></figcaption></figure>

The **KCFG** view might seem crowded. Let's break it into individual sections. Each section can be hidden using the hotkeys displayed at the bottom of the UI. Let's start with the left side.

{% hint style="warning" %}
The screenshots provided may not match your local KCFG exactly. It is normal for the node numbers to be different from the ones shown here. This section aims to provide context for understanding the KCFG output.
{% endhint %}

## Left section of KCFG

<figure><img src="../../.gitbook/assets/Left KCFG.png" alt=""><figcaption><p>Left section of the KCFG Visualizer</p></figcaption></figure>

The section on the left represents the **KCFG** of the execution in a minimal way, where nodes are linked together. Each node displays a summary of the state of the execution at that point. You can select a node by clicking on it, and the entire state will be displayed on the right. Once a node is selected, you can hide it from the **KCFG** using the hotkey `H`.

You can display all hidden nodes using the hotkey `H`. We’ll discuss that shortly. For now, let’s understand how to read a node. Notice the node highlighted in <mark style="color:yellow;">yellow</mark>. The first row `(61 steps)` represents the number of steps the prover has executed since the previous node.

The `9 (split)` indicates the node `id`, `9`, and the node type `(split)`. The main node types are:

* `init` - The initial node, or the root
* `leaf` - A node that has no children or successors
* `split` - A node that branches

Nodes can also be:

* `expanded` - A node that the node has been visited and processed
* `target` - A node representing the final state the prover aims to achieve
* `terminal` - A node that finished the execution of a `terminal rule`
* `frontier` - A node that has been discovered but not yet executed

Following the node `id`, there is a summary of the node:

* `k: JUMPI 151 bool2Word …` - represents the contents of the `<k>` cell and the point of execution at which the prover is in that node.
* `pc: 143` - represents the current value of the EVM program counter
* `callDepth: 1` - represents the current call stack depth ([more information here](https://docs.soliditylang.org/en/v0.8.17/security-considerations.html#call-stack-depth)).
* `statusCode: STATUSCODE:StatusCode` - shows the current status code. Here, `STATUSCODE` is the name of the symbolic variable, and `:StatusCode` shows the sort of the variable.
* `src: lib/forge-std/src/StdError.sol:16:16` - nodes can point to the Solidity source file, line, and column to which they belong.

A branching always follows a split node. In **KCFG'**s, branches are represented using nesting. In the highlighted node, the `k` field holds the value `k: JUMPI 151 bool2Word ( ( notBool VV0_x_114b9705:Int ==Int 12648430 ) )`. This indicates that the prover has identified a branching point. Here, `JUMPI` is an EVM opcode. `151` represents a jump destination, and `bool2Word ( notBool ( VV0_n_114b9705:Int ==Int 12648430 ) )` represents an equality check between the symbolic variable `VV0_n_114b9705` of sort `Int` and value `12648430` (the decimal value of `0xC0FFEE` from our `setNumber` function). The prover does not know if the `VV0` variable equals `12648430`, so it will branch and explore each possibility.

Below the highlighted section, you will see the following:

<figure><img src="../../.gitbook/assets/constraint.png" alt=""><figcaption></figcaption></figure>

The `constraint` keyword highlights the path the prover continues the execution. Nodes will be linked on this path as the exploration proceeds until either:

*   The execution of the branch is completed, and the `leaf` unifies with the target node.



    <figure><img src="../../.gitbook/assets/Leaf Unification.png" alt=""><figcaption><p>Leaf unification with the target node</p></figcaption></figure>
*   The execution of the branch halts.



    <figure><img src="../../.gitbook/assets/Execution Halted.png" alt=""><figcaption><p>Execution Halted</p></figcaption></figure>

## Right section of KCFG

You will see the status section at the top of the right section.

<figure><img src="../../.gitbook/assets/StatusSection.png" alt=""><figcaption><p>Status Section</p></figcaption></figure>

This section displays the selected node and any enabled or disabled options. You can show or hide views using the hotkeys displayed at the bottom of the screen. To exit the **KCFG,** you can use `Ctrl + C`.

<figure><img src="../../.gitbook/assets/image (14).png" alt=""><figcaption><p>Hotkeys</p></figcaption></figure>

### **The term view**

This section displays the entire state of the Ethereum virtual machine in the current node as a **K** configuration, with each property depicted in a cell. For example, the current hard fork will be displayed as `<schedule> SHANGHAI </schedule>` ([more about the configuration can be found here](https://jellopaper.org/evm/#configuration)). Using the hotkey `M`, you can minimize the term by hiding some cells in the configuration.

<figure><img src="../../.gitbook/assets/Termview.png" alt=""><figcaption><p>The term view</p></figcaption></figure>

### The constraint view

This section shows all the constraints that apply to the symbolic variables in the execution. In the unminimized term view, you can see that the cell `<caller>` contains the symbolic variable `CALLER_ID:Int`.

<figure><img src="../../.gitbook/assets/Screenshot 2023-05-12 at 10.40.48.png" alt=""><figcaption><p>The constraint view</p></figcaption></figure>

The constraint view indicates that `CALLER_ID` is an integer value greater than `0` and lower than `pow160` (that is $$2^{160}$$). These two constraints allow the prover to conclude that `CALLER_ID` could be any value within the Ethereum address range (between 0 and $$2^{160}$$). Similar constraints apply to `ORIGIN_ID` and `VV0_n_114b9705`. The last constraint states that the `VV1_inLuck_114b9705` symbolic variable can be either 1 or 0 since the `inLuck` variable is defined as `Bool` in the `setNumber` function.

When selecting the first node after the branching (`8`), a new constraint is added, indicating that `VV0_n_114b9705` is not equal on this branch to `12648430`.

The constraint view can be enabled or disabled using the hotkey `c`.

### The custom view

This section displays the Solidity source code, highlighting the currently executing lines. If data is not available, a message will be displayed.

<figure><img src="../../.gitbook/assets/Custom View.png" alt=""><figcaption><p>The custom viewThe custom view</p></figcaption></figure>

The custom view can be enabled or disabled using the hotkey `V`.

## Investigating the failed proof

Now, let’s find out why the proof is failing. At the top of the term view, highlighted in <mark style="color:yellow;">yellow</mark>, we can see the `#halt` production in the `<k>` cell, indicating that the test execution has finished. Additionally, highlighted in <mark style="color:red;">red</mark>, the status code as `EVMC_REVERT`, meaning that the transaction has been reverted. At this point, we can look in the constraint view and identify that our function arguments, highlighted in <mark style="color:orange;">orange</mark>, are exactly the ones required to trigger the revert in `setFunction`.

<figure><img src="../../.gitbook/assets/Last Node.png" alt=""><figcaption><p>Examining the last node</p></figcaption></figure>
