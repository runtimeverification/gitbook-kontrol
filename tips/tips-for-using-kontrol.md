---
description: Some Tips for using Kontrol from Developers at RV
---

# Tips for using Kontrol

### Running a proof with Kontrol

The following is a suggested process to follow when running a proof with **Kontrol:**

1. Run the proof with `no-break-on-calls`, `--max-depth 10000` and `--max-iterations 5` or `10`
2. Inspect the `kcfg` for branching.
3. If there is branching, check if the branching condition is `true` or `false`:
   * Try to figure out the simplifications to discharge it. To do this efficiently, one needs to be familiar existing simplifications.
   * Add the simplifications and create a claim in the form `runLemma => doneLemma` that demonstrably simplifies the branching condition.
   * Remove the branching node from the `kcfg` using `foundry-remove-node`
   * Repeat from step 1.
4. Repeat from step 1.

When writing a claim using the `runLemma-doneLemma` pattern to check if an expression simplifies, it's important to remember that running the claim includes an `implication check`. This means that if your claim is in the form of `runLemma(A) => doneLemma(B)` and it passes, it doesn't guarantee that `A` fully simplifies to `B`. Instead, it might simplify to an expression `B'` that implies `B`.

***

### Decoding KEVM expressions

* Solidity uses bitwise expressions, such as `maxUInt160 &Int X`, to extract a variable with a specific number of bits from a larger word. The number of bits can often provide a clue about the type of the variable. For example, `maxUInt160` typically represents an address, while `maxUInt8` represents a `boolean` value.
* When using the `symbolicStorage` cheatcode, you may encounter expressions like `#lookup(?STORAGE0:Map, 6)`. This expression accesses storage slot 6 of the symbolic storage represented by the `STORAGE0` variable. If you want to determine which storage variable this expression corresponds to, you can follow these steps:&#x20;
  * First, ascertain the contract that `STORAGE0` corresponds to.
    * The first call of `symbolicStorage` creates the symbolic variable `STORAGE`, followed by `STORAGE0`, `STORAGE1`, `STORAGE2`, `STORAGE3`, and so on. Therefore, you can use the order in which `symbolicStorage` was called in each contract to map each variable to its contract.
    * Another option is to check the `<accounts>` cell in the `KEVM` configuration. In each `<account>`, the `<acctId>` cell contains the address of the contract, and the `<storage>` cell contains the storage. If you know the address of each contract, you can map it to the storage variable.
  * Next, determine which variable corresponds to storage slot 6.
    * The easiest way to do this is by calling `forge inspect ContractName` storage, where `ContractName` represents the contract identified in the previous step. This command will output a `JSON` result, with the storage field containing a list of all storage slots in the contract. The label of each slot corresponds to the name of the storage variable.
  * Some storage slots contain more than one variable at different offsets. If your expression is, for example, `#lookup ( ?STORAGE0:Map , 6 ) >>Int 8`, this means it is offsetting the storage slot by 8 bits, or 1 byte. In the previous step, you should look for the variable at that storage slot with offset 1.

***

### Forge Debugger with Kontrol

If you need to understand why the memory looks a certain way at a certain point during execution, you can use the [Forge debugger](https://book.getfoundry.sh/forge/debugger).&#x20;

{% hint style="info" %}
If you decided to use Forge debugger. You may need to create a version of the test with concrete values instead of symbolic ones.
{% endhint %}

The debugger can be used to set breakpoints and step through the EVM code to observe how the memory changes. This can be particularly useful to understand details about the [Solidity memory layout](https://docs.soliditylang.org/en/latest/internals/layout\_in\_memory.html) that may not be well-documented.

***

### Other Proving and Debugging Tips

* If `kontrol foundry-prove` hangs during an execute step (no response for hours) **or** it crashes because `kore-rpc` returned an empty response, it may be caused by the configuration becoming too large.&#x20;
  * To troubleshoot this issue, inspect the node that was being extended using `foundry-show` or `foundry-view-kcfg`. Check if there are any abnormally large expressions in any of the cells and consider writing lemmas to simplify them.
* If you are using the `infiniteGas` cheatcode and the expression in the `<gas>` or `<callGas>` cell is growing out of control without being simplified, you can call `kontrol foundry-prove` with the `--auto-abstract-gas` option. This will automatically abstract the gas expression into a symbolic variable. Only do this if you are **not** concerned with measuring gas consumption, as you will lose that information. If possible, write lemmas to simplify the gas expression instead.
* Some instructions in `KEVM` may cause branching based on whether a symbolic address already exists in the `<accounts>` cell or if it is a new address. This situation occurs when the `prank` cheatcode is called with a symbolic address. You can refer to [this issue](https://github.com/runtimeverification/evm-semantics/issues/1752#issuecomment-1601611907) in `foundry-show` for an example.
  * To resolve this issue, the usual solution is to add one `vm.assume(symbolicAddress != ...)` for each of the preexisting addresses. These addresses should correspond to:
    * the test contract address
    * the cheatcodes contract address
    * the address of any other contracts deployed within the test.
* The `&&` and `||` operators are short-circuit operators. They introduce branching when evaluated. This means that if you have a line like `vm.assume(p && q)` or `vm.assume(p || q)`, it will introduce branching in the execution, which may not be immediately obvious.
* When adding a new lemma to remove an unnecessary branch, be sure to delete the `split` node from the `KCFG` before continuing. Otherwise, both branches will still exist, but the unnecessary one will simplify to `#Bottom`.
