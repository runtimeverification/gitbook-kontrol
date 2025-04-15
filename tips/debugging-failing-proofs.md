---
description: Tips and tricks for running Kontrol and debugging failing proofs
---

# Debugging Failing Proofs

When a proof fails in Kontrol, there are several strategies you can use to debug and potentially resolve the issue. Here are some common approaches:

## Inspecting the KCFG

1. Inspect the KCFG for branching.
2. If there is branching, check if the branching condition is `true` or `false`:
   * Try to figure out the simplifications to discharge it, as described in as described in the [KEVM Lemmas](..guides/advancing-proofs/kevm-lemmas.md) section. To do this efficiently, one needs to be familiar with existing simplifications.
   * Add the simplifications and create a claim in the form `runLemma => doneLemma` that demonstrably simplifies the branching condition.
   * Remove the branching node from the `kcfg` using `kontrol remove-node`
   * Rerun Kontrol and repeat from step 1.

When writing a claim using the `runLemma-doneLemma` pattern to check if an expression simplifies, it's important to remember that running the claim includes an `implication check`. This means that if your claim is in the form of `runLemma(A) => doneLemma(B)` and it passes, it doesn't guarantee that `A` fully simplifies to `B`. Instead, it might simplify to an expression `B'` that implies `B`.

### Decoding KEVM expressions

The following tips might be useful when inspecting branching conditions or nodes in the `kcfg`. If you don't know where an expression comes from, this might help figuring out what they mean and what part of the Solidity code they correspond to:

* Solidity uses bitwise expressions, such as `maxUInt160 &Int X`, to extract a variable with a specific number of bits from a larger word. The number of bits can often provide a clue about the type of the variable. For example, `maxUInt160` typically represents an address, while `maxUInt8` represents a `boolean` value.
* When using the `symbolicStorage` cheatcode, you may encounter expressions like `#lookup(?STORAGE0:Map, 6)`. This expression accesses storage slot 6 of the symbolic storage represented by the `STORAGE0` variable. If you want to determine which storage variable this expression corresponds to, you can follow these steps:&#x20;
  * First, ascertain the contract that `STORAGE0` corresponds to.
    * The first call of `symbolicStorage` creates the symbolic variable `STORAGE`, followed by `STORAGE0`, `STORAGE1`, `STORAGE2`, `STORAGE3`, and so on. Therefore, you can use the order in which `symbolicStorage` was called in each contract to map each variable to its contract.
    * Another option is to check the `<accounts>` cell in the `KEVM` configuration. In each `<account>`, the `<acctId>` cell contains the address of the contract, and the `<storage>` cell contains the storage. If you know the address of each contract, you can map it to the storage variable.
  * Next, determine which variable corresponds to storage slot 6.
    * The easiest way to do this is by calling `forge inspect ContractName storage`, where `ContractName` represents the contract identified in the previous step. This command will output a `JSON` result, with the `storage` field containing a list of all storage slots in the contract. The label of each slot corresponds to the name of the storage variable.
  * Some storage slots contain more than one variable at different offsets. If your expression is, for example, `#lookup ( ?STORAGE0:Map , 6 ) >>Int 8`, this means it is offsetting the storage slot by 8 bits, or 1 byte. In the previous step, you should look for the variable at that storage slot with offset 1.

### Use Debugger

If you need to understand why the memory looks a certain way at a certain point during execution, you can use [Simbolik](https://simbolik.runtimeverification.com/) or [Forge](https://book.getfoundry.sh/forge/debugger) debuggers.&#x20;

{% hint style="info" %}
To use the debugger, you may need to create a version of the test with concrete values instead of symbolic ones.
{% endhint %}

The debugger can be used to set breakpoints and step through the EVM code to observe how the memory changes. This can be particularly useful to understand details about the [Solidity memory layout](https://docs.soliditylang.org/en/latest/internals/layout\_in\_memory.html) that may not be well-documented.

## Adjusting SMT Solver Settings

### Increasing Timeout
If the proof is timing out, you can increase the SMT solver timeout:

```bash
kontrol prove --smt-timeout 5000  # Timeout in milliseconds
```

The default timeout is 1000ms. Increasing this value gives the solver more time to find a solution, but be aware that it will make the verification process slower.

### Changing SMT Tactics
Different SMT tactics can be more effective for different types of proofs:

```bash
# Use qfnra-nlsat tactic (good for non-linear arithmetic)
kontrol prove --smt-tactic '(check-sat-using qfnra-nlsat)'

# Use default smt tactic
kontrol prove --smt-tactic '(check-sat-using smt)'
```

Experiment with different tactics to find what works best for your specific proof.

## Handling Overflows

### Using Assumptions
For arithmetic operations that might overflow, you can add assumptions to constrain the values:

```solidity
function testNoOverflow() public {
    uint256 x = vm.freshUInt(256, "x");
    uint256 y = vm.freshUInt(256, "y");
    
    // Add assumption to prevent overflow
    vm.assume(x <= type(uint256).max - y);
    
    uint256 sum = x + y;
    // Your assertions here
}
```

### Using Unchecked Blocks
For operations where overflow is expected or acceptable, use unchecked blocks:

```solidity
function testWithOverflow() public {
    uint256 x = vm.freshUInt(256, "x");
    uint256 y = vm.freshUInt(256, "y");
    
    unchecked {
        uint256 sum = x + y;
        // Your assertions here
    }
}
```

### Using K Lemmas
For more complex cases, you can explore defining K lemmas as described in [Advancing Proofs](..guides/advancing-proofs/kevm-lemmas.md).

## Other Debugging Tips

### Other Proving and Debugging Tips



1. **Simplify the Proof**

   If `kontrol prove` hangs during an execute step (no response for hours) **or** it crashes because `kore-rpc` returned an empty response, it may be caused by the configuration becoming too large.&#x20;
   - Use `kontrol show` or `kontrol view-kcfg` to check if nodes have abnormally large expressions in any of the cells and consider writing lemmas to simplify them
   - Break down complex proofs into smaller, more manageable parts
   - Verify individual components before combining them
   - Use concrete values for some variables to reduce complexity

2. **Check Path Conditions**
   - Use `kontrol view-kcfg` to inspect the control flow graph
   - Look for unexpected branching conditions
   - Verify that all paths are being explored as expected
   - Check for branching on whether a symbolic address is in the `<accounts>` cell (an example is shown in [this issue](https://github.com/runtimeverification/evm-semantics/issues/1752#issuecomment-1601611907)):
        * This can be resolved by adding one `vm.assume(symbolicAddress != ...)` for each of the preexisting addresses. These addresses should correspond to:
            * the test contract address
            * the cheatcodes contract address
            * the address of any other contracts deployed within the test.
   - Look for branching caused by short-circuit operators such as `&&` and `||`
      * While using these operators shouldn't be causing a failure, they introduce branching when evaluated

3. **Reduce Symbolic Variables**
   - Limit the range of symbolic variables
   - Add more constraints to reduce the search space

4. **Use Lemmas**
   - Create and prove lemmas for complex properties and arithmetic expressions
   - Use lemmas to break down the proof into smaller steps
   - When adding a new lemma to remove an unnecessary branch, be sure to delete the `split` node from the `KCFG` before continuing. Otherwise, both branches will still exist, but the unnecessary one will simplify to `#Bottom`

5. **Check Storage Layout**
   - Verify that storage slots are being accessed correctly
   - Ensure that storage updates are happening in the expected order
   - Check for potential storage collisions

Debugging formal verification proofs often requires a combination of these approaches. Start with the simplest solution and gradually move to more complex ones if needed. 