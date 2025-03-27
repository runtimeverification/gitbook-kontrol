# Node Refutation

This document walks through how to use the new commands added to **Kontrol** to deal with undesired branches during symbolic execution. The commands that we'll be introducing are the following:

`kontrol refute-node`: Marks a node as refuted, which pauses execution in that branch, and creates a refutation subproof stating that the branch is infeasible.

`kontrol unrefute-node`: Removes the refuted status of a node, allowing execution to continue in that branch.

`kontrol split-node`: Creates a split on a node using a branching condition supplied by the user.

## Example

We'll be using the following running example, which can also be found as one of the tests in `PrankTest.t.sol` in the **Kontrol** repo ([link](https://github.com/runtimeverification/kontrol/blob/master/src/tests/integration/test-data/foundry/test/PrankTest.t.sol)):

```solidity
// SPDX-License-Identifier: UNLICENSED                                                                                                                                                                      
pragma solidity =0.8.13;

import "forge-std/Test.sol";

contract Prank {
    function msgSender() external returns (address) {
        return msg.sender;
    }
}

contract PrankTest is Test {
    Prank prankContract;

    function setUp() public {
        prankContract = new Prank();
    }

    function testSymbolicStartPrank(address addr) public {
        vm.startPrank(addr);
        assert(prankContract.msgSender() == addr);
        vm.stopPrank();
    }
}
```

`testSymbolicStartPrank` receives a symbolic address `addr` as parameter and uses `startPrank/stopPrank` to impersonate this address when calling `prankContract.msgSender()`. The test simply asserts that the `msg.sender` observed by `prankContract` is in fact the symbolic addressed we are pranking on.

To build, run the command below:&#x20;

```bash
kontrol build 
```

To run the test use the following:

```bash
kontrol prove --match-test PrankTest.testSymbolicStartPrank
```

&#x20;This will pass, as expected. However, if we display the **KCFG** of this test by running:

```bash
kontrol show PrankTest.testSymbolicStartPrank
```

We will see that it looks like this:

```bash
┌─ 1 (root, init)
│   k: #execute ~> CONTINUATION:K
│   pc: 0
│   callDepth: 0
│   statusCode: STATUSCODE:StatusCode
│   src: test/PrankTest.t.sol:11:24
│
.
.
.
│  (1 step)
├─ 5
│   k: #execute ~> CONTINUATION:K
│   pc: 0
│   callDepth: 0
│   statusCode: STATUSCODE:StatusCode
│   src: test/PrankTest.t.sol:11:24
│
│  (658 steps)
├─ 7
│   k: #loadAccount VV0_addr_114b9705:Int ~> #setPrank VV0_addr_114b9705:Int .Account f ...
│   pc: 1055
│   callDepth: 0
│   statusCode: STATUSCODE:StatusCode
│   src: lib/forge-std/src/StdInvariant.sol:73:74
┃
┃ (1 step)
┣━━┓
┃  │
┃  ├─ 8
┃  │   k: #accessAccounts 728815563385977040452943777879061427756277306518 ~> #setPrank 72 ...
┃  │   pc: 1055
┃  │   callDepth: 0
┃  │   statusCode: STATUSCODE:StatusCode
┃  │   src: lib/forge-std/src/StdInvariant.sol:73:74
┃  │
┃  .
┃  .
┃  .
┃
┣━━┓
┃  │
┃  ├─ 9
┃  │   k: #accessAccounts 645326474426547203313410069153905908525362434349 ~> #setPrank 64 ...
┃  │   pc: 1055
┃  │   callDepth: 0
┃  │   statusCode: STATUSCODE:StatusCode
┃  │   src: lib/forge-std/src/StdInvariant.sol:73:74
┃  │
┃  .
┃  .
┃  .
┃
┣━━┓
┃  │
┃  ├─ 10
┃  │   k: #accessAccounts 491460923342184218035706888008750043977755113263 ~> #setPrank 49 ...
┃  │   pc: 1055
┃  │   callDepth: 0
┃  │   statusCode: STATUSCODE:StatusCode
┃  │   src: lib/forge-std/src/StdInvariant.sol:73:74
┃  │
┃  .
┃  .
┃  .
┃
┗━━┓
   │
   ├─ 11
   │   k: #newAccount VV0_addr_114b9705:Int ~> #accessAccounts VV0_addr_114b9705:Int ~> #s ...
   │   pc: 1055
   │   callDepth: 0
   │   statusCode: STATUSCODE:StatusCode
   │   src: lib/forge-std/src/StdInvariant.sol:73:74
   │
   .
   .
   .
```

{% hint style="warning" %}
Note: All KCFG code blocks will have some nodes that have been omitted for succinctness.
{% endhint %}

As we can see, there is a non-deterministic branch on `node 7`. This happens because when we prank on an address, **Kontrol** needs to know whether this address already exists in the `<accounts>` cell or if it's a new address.&#x20;

As a result, it creates one branch (`node 11`) for the case where the address is new, and three other branches corresponding to the cases where `addr` is one of the three addresses already present in the `<accounts>`  cell:&#x20;

* `prankContract` : `node 10`
* the cheatcodes contract `vm`: `node 9`
* the `PrankTest` contract itself: `node 8`

This is not a big problem for a small example like this, but in real-world contracts, this non-deterministic branching has the potential to increase by several times the running time of symbolic execution, as the same test will have to execute in each branch, and in most cases the specific address is unlikely to make a difference.

We usually solve this problem by adding an assumption to the test that the address being pranked on is not one of these special addresses:

```solidity
function testSymbolicStartPrank(address addr) public {
    vm.assume(addr != address(this));
    vm.assume(addr != address(vm));
    vm.assume(addr != address(prankContract))

    vm.startPrank(addr);
    assert(prankContract.msgSender() == addr);
    vm.stopPrank();
}
```

However, changing the source code forces us to restart the test from the beginning, which in a real-world example might also cost significant time. Instead, this tutorial will demonstrate how to use the new **Kontrol** commands to control the execution of individual branches, pausing exploration in the branches we don't care about and continuing to explore only the branches that we do. Then, after we have ensured that the desired branches pass, we can add the assumptions and restart the test.

## Turning non-deterministic branches into splits

Currently, `kontrol refute-node` doesn't work with non-deterministic branches, so first we need to turn the non-deterministic branch into a split on a branching condition. We can do this with `kontrol split-node`.

First, we use `kontrol remove-node` to remove the root of the non-deterministic branch. We can do this by running:

```bash
kontrol remove-node PrankTest.testSymbolicStartPrank 7 
```

Our **KCFG** now looks like this:

```bash
┌─ 1 (root, init)
│   k: #execute ~> CONTINUATION:K
│   pc: 0
│   callDepth: 0
│   statusCode: STATUSCODE:StatusCode
│   src: test/PrankTest.t.sol:11:24
│
.
.
.
│
│  (1 step)
└─ 5 (leaf, pending)
    k: #execute ~> CONTINUATION:K
    pc: 0
    callDepth: 0
    statusCode: STATUSCODE:StatusCode
    src: test/PrankTest.t.sol:11:24
```

Next, we use `kontrol split-node` to insert the branching condition that we want to split on. In this case, we first want to branch on whether `addr` equals the address of the `PrankTest` contract. To do this you will run:

```bash
> kontrol split-node PrankTest.testSymbolicStartPrank 5 "VV0_addr_114b9705 ==Int 728815563385977040452943777879061427756277306518"
Node 5 has been split into [20, 21] on condition VV0_addr_114b9705 ==Int 728815563385977040452943777879061427756277306518.
```

Now, our **KCFG** now looks like this:

```bash
┌─ 1 (root, init)
│   k: #execute ~> CONTINUATION:K
│   pc: 0
│   callDepth: 0
│   statusCode: STATUSCODE:StatusCode
│   src: test/PrankTest.t.sol:11:24
│
.
.
.
│
│  (1 step)
├─ 5 (split)
│   k: #execute ~> CONTINUATION:K
│   pc: 0
│   callDepth: 0
│   statusCode: STATUSCODE:StatusCode
│   src: test/PrankTest.t.sol:11:24
┃
┃ (branch)
┣━━┓ constraint: VV0_addr_114b9705:Int ==Int 728815563385977040452943777879061427756277306518
┃  │
┃  └─ 20 (leaf, pending)
┃      k: #execute ~> CONTINUATION:K
┃      pc: 0
┃      callDepth: 0
┃      statusCode: STATUSCODE:StatusCode
┃      src: test/PrankTest.t.sol:11:24
┃
┗━━┓ constraint: ( notBool VV0_addr_114b9705:Int ==Int 728815563385977040452943777879061427756277306518 )
   │
   └─ 21 (leaf, pending)
       k: #execute ~> CONTINUATION:K
       pc: 0
       callDepth: 0
       statusCode: STATUSCODE:StatusCode
       src: test/PrankTest.t.sol:11:24
```

We can now repeat by further splitting into the other two addresses. You will need to run the following:

```bash
> kontrol split-node PrankTest.testSymbolicStartPrank 21 "VV0_addr_114b9705 ==Int 645326474426547203313410069153905908525362434349"
Node 21 has been split into [22, 23] on condition VV0_addr_114b9705 ==Int 645326474426547203313410069153905908525362434349.
> kontrol split-node PrankTest.testSymbolicStartPrank 23 "VV0_addr_114b9705 ==Int 491460923342184218035706888008750043977755113263"
Node 23 has been split into [24, 25] on condition VV0_addr_114b9705 ==Int 491460923342184218035706888008750043977755113263.
```

Finally our **KCFG** will look like this:

```bash
┌─ 1 (root, init)
│   k: #execute ~> CONTINUATION:K
│   pc: 0
│   callDepth: 0
│   statusCode: STATUSCODE:StatusCode
│   src: test/PrankTest.t.sol:11:24
│
.
.
.
│
│  (1 step)
├─ 5 (split)
│   k: #execute ~> CONTINUATION:K
│   pc: 0
│   callDepth: 0
│   statusCode: STATUSCODE:StatusCode
│   src: test/PrankTest.t.sol:11:24
┃
┃ (branch)
┣━━┓ constraint: VV0_addr_114b9705:Int ==Int 728815563385977040452943777879061427756277306518
┃  │
┃  └─ 20 (leaf, pending)
┃      k: #execute ~> CONTINUATION:K
┃      pc: 0
┃      callDepth: 0
┃      statusCode: STATUSCODE:StatusCode
┃      src: test/PrankTest.t.sol:11:24
┃
┗━━┓ constraint: ( notBool VV0_addr_114b9705:Int ==Int 728815563385977040452943777879061427756277306518 )
   │
   ├─ 21 (split)
   │   k: #execute ~> CONTINUATION:K
   │   pc: 0
   │   callDepth: 0
   │   statusCode: STATUSCODE:StatusCode
   │   src: test/PrankTest.t.sol:11:24
   ┃
   ┃ (branch)
   ┣━━┓ constraint: VV0_addr_114b9705:Int ==Int 645326474426547203313410069153905908525362434349
   ┃  │
   ┃  └─ 22 (leaf, pending)
   ┃      k: #execute ~> CONTINUATION:K
   ┃      pc: 0
   ┃      callDepth: 0
   ┃      statusCode: STATUSCODE:StatusCode
   ┃      src: test/PrankTest.t.sol:11:24
   ┃
   ┗━━┓ constraint: ( notBool VV0_addr_114b9705:Int ==Int 645326474426547203313410069153905908525362434349 )
      │
      ├─ 23 (split)
      │   k: #execute ~> CONTINUATION:K
      │   pc: 0
      │   callDepth: 0
      │   statusCode: STATUSCODE:StatusCode
      │   src: test/PrankTest.t.sol:11:24
      ┃
      ┃ (branch)
      ┣━━┓ constraint: VV0_addr_114b9705:Int ==Int 491460923342184218035706888008750043977755113263
      ┃  │
      ┃  └─ 24 (leaf, pending)
      ┃      k: #execute ~> CONTINUATION:K
      ┃      pc: 0
      ┃      callDepth: 0
      ┃      statusCode: STATUSCODE:StatusCode
      ┃      src: test/PrankTest.t.sol:11:24
      ┃
      ┗━━┓ constraint: ( notBool VV0_addr_114b9705:Int ==Int 491460923342184218035706888008750043977755113263 )
         │
         └─ 25 (leaf, pending)
             k: #execute ~> CONTINUATION:K
             pc: 0
             callDepth: 0
             statusCode: STATUSCODE:StatusCode
             src: test/PrankTest.t.sol:11:24
```

Note that the `leaf` nodes correspond to the same cases as the non-deterministic branch we had previously.

## Refuting Undesirable Branches

Now that the non-deterministic branch has been turn into a split, we can use `kontrol refute-node` to refute the nodes we don't want to consider. We start with `node 20`, where `addr` equals the address of the `PrankTest` contract:

```bash
> kontrol refute-node PrankTest.testSymbolicStartPrank 20
WARNING 2024-03-23 17:13:49,385 pyk.proof.implies - Building a RefutationProof that has known soundness issues: See https://github.com/runtimeverification/haskell-backend/issues/3605.

Claim for the refutation:

claim [refuted-20]: ( VV0_addr_114b9705:Int ==Int 728815563385977040452943777879061427756277306518 => false )
  requires ( 0 <=Int VV0_addr_114b9705:Int
   andBool ( VV0_addr_114b9705:Int <Int pow160
           ))
  [label(refuted-20)]
```

{% hint style="info" %}
`kontrol refute-node` builds a refutation subproof that the refuted branch is infeasible, which the log message warns has soundness issues.&#x20;
{% endhint %}

Currently, this is irrelevant because there isn't a way to discharge subproofs in **Kontrol**. Instead, refuted branches need to be eliminated by adding new lemmas that make them vacuous, or by restarting the proof with additional assumptions, as in our case.

{% hint style="info" %}
&#x20;Having `kontrol refute-node` print the claim above depends on an as-of-yet unmerged [PR](https://github.com/runtimeverification/kontrol/pull/477).
{% endhint %}

This claim always has the form `claim X => false requires ...` and represents the refutation proof (`X` is the branch condition for the refuted branch, and the `requires` clauses represent all path conditions that it might depend on). This is useful if we are planning on disproving this branch by adding a lemma, as the generated claim can serve as a test that the lemma works and can easily be upstreamed into **Kontrol**.

We can now refute the other two undesirable branches, leaving us with only `node 25`, representing the case where `addr` is different from all of the special addresses:

```bash
kontrol refute-node PrankTest.testSymbolicStartPrank 22  
```

```bash
kontrol refute-node PrankTest.testSymbolicStartPrank 24
```

If we inspect the **KCFG** with **Kontrol** show, we can observe that nodes `20`, `21` and `24` are now marked as `refuted` instead of `pending`:

```bash
┃ (branch)
┣━━┓ constraint: VV0_addr_114b9705:Int ==Int 728815563385977040452943777879061427756277306518
┃  │
┃  └─ 20 (leaf, refuted)
┃      k: #execute ~> CONTINUATION:K
┃      pc: 0
┃      callDepth: 0
┃      statusCode: STATUSCODE:StatusCode
┃      src: test/PrankTest.t.sol:11:24
```

This means that execution will not continue from them when we resume the proof.

## Resuming the Proof

Let's now resume the proof:

```bash
> kontrol prove --match-test PrankTest.testSymbolicStartPrank --verbose
...
PROOF FAILED: test%PrankTest.testSymbolicStartPrank(address):0
time: 3m 11s
The proof cannot be completed while there are refuted nodes: [20, 22, 24].
Either unrefute the nodes or discharge the corresponding refutation subproofs.
```

As we can see, the proof has failed because there are branches that haven't been executed due to being refuted. If we use `kontrol show`, however, we can see that the branch we care about has reached the target node, as expected:

```bash
  ┗━━┓ constraint: ( notBool VV0_addr_114b9705:Int ==Int 491460923342184218035706888008750043977755113263 )
     │
     ├─ 25
     │   k: #execute ~> CONTINUATION:K
     │   pc: 0
     │   callDepth: 0
     │   statusCode: STATUSCODE:StatusCode
     │   src: test/PrankTest.t.sol:11:24
     │
     .
     .
     .
     │
     │  (656 steps)
     ├─ 28 (terminal)
     │   k: #halt ~> CONTINUATION:K
     │   pc: 194
     │   callDepth: 0
     │   statusCode: EVMC_SUCCESS
     │   src: lib/forge-std/lib/ds-test/src/test.sol:37:38
     │
     ┊  constraint: OMITTED CONSTRAINT
     ┊  subst: OMITTED SUBST
     └─ 6 (leaf, target, terminal)
         k: #halt ~> CONTINUATION:K
         pc: PC_CELL_5d410f2a:Int
         callDepth: CALLDEPTH_CELL_5d410f2a:Int
         statusCode: STATUSCODE_FINAL:StatusCode
```

Now that we have finished execution of our desired branch, if we wished we could explore the other branches by using `kontrol unrefute-node` to turn them back from `refuted` to `pending`:

```bash
kontrol unrefute-node PrankTest.testSymbolicStartPrank 20
```

```bash
kontrol unrefute-node PrankTest.testSymbolicStartPrank 22
```

```bash
kontrol unrefute-node PrankTest.testSymbolicStartPrank 24
```

Or we can finally add the assumptions to the test as we had planned, so that we avoid these branches entirely, knowing that the remaining branch is guaranteed to pass.

## Summary

In summary, the following workflow can be used when a user runs into undesired branching:

1. If the branching is non-deterministic, convert it into a split by using `kontrol remove-node` and `kontrol split-node` on the conditions the user cares about.&#x20;
   1. Obs.: If this case is general, rather than specific to this particular proof, consider adding it as a special branch pattern in KEVM so that this non-deterministic branch gets automatically turned into a split.&#x20;
2. Use `kontrol refute-node` to pause execution of the undesired branch(es).&#x20;
   1. Obs.: In case of a mistake, `kontrol unrefute-node` can be used to revert this.&#x20;
3. Continue execution on the desired branches.&#x20;
4. After execution of the desired branches terminates, handle the undesired branches:&#x20;
   1. If the branch was the result of missing lemmas, add these lemmas and either restart the proof or unrefute the node (in the latter case, the branching will remain but the invalid branch will be marked as vacuous). Upstream the `claim` generated by `kontrol refute-node` as a test for the new lemmas.&#x20;
   2. If the branch was the result of missing assumptions, add these assumptions to the test and restart the proof.
