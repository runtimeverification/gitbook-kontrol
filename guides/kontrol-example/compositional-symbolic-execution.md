---
description: Learn about Compositional Symbolic Execution (CSE) and optimization techniques in Kontrol
---

# Compositional Symbolic Execution (CSE)

Compositional Symbolic Execution (CSE) is a powerful optimization technique used in Kontrol to improves verification performance by analyzing functions in isolation and creating reusable summaries. This approach minimizes path traversal and makes verification more efficient, especially for complex smart contracts with multiple external dependencies.

## What is CSE?

CSE addresses a fundamental challenge in symbolic execution: when a function calls external dependencies, the verification tool must explore all possible execution paths through those dependencies, leading to exponential path explosion and slow verification times.

CSE solves this by:

1. **Analyzing functions in isolation**: External functions are executed in a more abstract, symbolic setting
2. **Generating function summaries**: Results are stored as K rules that capture the function's behavior
3. **Reusing summaries**: When the function is called again, the summary is used instead of re-executing
4. **Higher priority application**: Summaries are inserted with higher priority to ensure they're used first, otherwise the tool will explore all possible execution paths through the dependencies in a regular symbolic execution manner

## How CSE Works

### Traditional Symbolic Execution
Without CSE, when a test calls a function that makes external calls:
```
Test Function → External Function A → External Function B
                ↓ (explore all paths)
              Multiple execution paths
                ↓ (explore all paths)  
            Even more execution paths
```

### With CSE
With CSE enabled, the process becomes:
```
1. Analyze External Function B in isolation → Generate Summary B
2. Analyze External Function A (using Summary B) → Generate Summary A  
3. Execute Test Function (using Summary A) → Complete verification
```

## Example: Simple Arithmetic Contract

Throughout this guide, we'll use a simple arithmetic contract to demonstrate CSE, node merging, and NatSpec preconditions. The contract implements two functions: `add` and `addAndIncrement`. The `add` function adds two numbers and returns the result. The `addAndIncrement` function adds two numbers and returns the result, and if the `shouldIncrement` flag is `true`, it increments the result by 1.

```solidity
// src/Arithmetic.sol
pragma solidity ^0.8.13;

contract Arithmetic {
    function add(uint256 a, uint256 b) external pure returns (uint256) {
        return a + b;
    }
    
    function addAndIncrement(uint256 a, uint256 b, bool shouldIncrement) external pure returns (uint256) {
        uint256 result = a + b;
        
        if (shouldIncrement) {
            result = result + 1; // Increment by 1
        }
        
        // All paths converge here for final validation
        assert(result >= a + b);
        return result;
    }
}

// test/ArithmeticTest.t.sol
pragma solidity ^0.8.13;

import {Test} from "forge-std/Test.sol";
import {Arithmetic} from "../src/Arithmetic.sol";

contract ArithmeticTest is Test {
    Arithmetic public arithmetic;
    
    function setUp() public {
        arithmetic = new Arithmetic();
    }
    
    function testAdd(uint256 a, uint256 b) public {
        uint256 result = arithmetic.add(a, b);
        assert(result == a + b);
    }
    
    function testAddWithAssumptions(uint256 a, uint256 b) public {
        // Prevent overflow by assuming the values are within the range of a uint128
        vm.assume(a <= 2 ** 128 - 1);
        vm.assume(b <= 2 ** 128 - 1);
        
        uint256 result = arithmetic.add(a, b);
        assert(result == a + b);
    }
    
    function testAddAndIncrement(uint256 a, uint256 b, bool shouldIncrement) public {
        vm.assume(a <= 2 ** 128 - 1);
        vm.assume(b <= 2 ** 128 - 1);
        
        uint256 result = arithmetic.addAndIncrement(a, b, shouldIncrement);
        assert(result >= a + b);
    }
}
```

## Running CSE

To use CSE, simply add the `--cse` flag to your `kontrol prove` command:

```bash
# Build the project
kontrol build

# Run with CSE enabled
kontrol prove --match-test ArithmeticTest.testAdd --cse
```

As you will notice in the logs, when CSE is enabled, Kontrol will:
1. First symbolically execute `add` and `addAndIncrement` functions in isolation to generate a summary for each function
2. Then execute the test using the `add` summary, instead of symbolically executing `add` when the test calls it

Using CSE, therefore, eliminates the need to re-execute `add` for every path in the test, improving performance. This summary can be reused for other tests that call `add`, and is particularly useful for tests that call `add` multiple times.

{% hint style="info" %}
**Note**: This simple `add` example is meant to illustrate the CSE concept, but CSE is most beneficial in practice when dealing with complex functions that have multiple external dependencies, expensive computations, or complex branching logic. In real-world scenarios, CSE shines when verifying contracts that interact with multiple external contracts or have intricate business logic that would otherwise create exponential path explosion.
{% endhint %}

### Visualizing the KCFG

You can visualize the KCFG (K Control Flow Graph) to see how CSE affects the proof structure. For example, for the `testAdd` function, you can run:

```bash 
kontrol show 'test%Arithmetic.add('
```
The resulting KCFG will look like this:
```
┌─ 1 (root, split, init)
│   k: #execute ~> CONTINUATION:K
│   pc: 0
│   callDepth: CALLDEPTH_CELL:Int
│   statusCode: STATUSCODE:StatusCode
│   src: test/Arithmeric.t.sol:3:20
│   method: test%Arithmetic.add(uint256,uint256)
┃
┃ (branch)
┣━━┓ subst: .Subst
┃  ┃ constraint:
┃  ┃     KV0_a:Int <=Int ( maxUInt256 -Int KV1_b:Int )
┃  │
┃  ├─ 8
┃  │   k: #execute ~> CONTINUATION:K
┃  │   pc: 0
┃  │   callDepth: CALLDEPTH_CELL:Int
┃  │   statusCode: STATUSCODE:StatusCode
┃  │   src: test/Arithmeric.t.sol:3:20
┃  │   method: test%Arithmetic.add(uint256,uint256)
┃  │
┃  │  (1279 steps)
┃  ├─ 6 (terminal)
┃  │   k: #halt ~> CONTINUATION:K
┃  │   pc: 154
┃  │   callDepth: CALLDEPTH_CELL:Int
┃  │   statusCode: EVMC_SUCCESS
┃  │   src: test/Arithmeric.t.sol:4:6
┃  │   method: test%Arithmetic.add(uint256,uint256)
┃  │
┃  ┊  constraint:
┃  ┊      ( notBool <acctID>
  C_ARITHMETIC_ID:Int
</acctID> in_keys ( ACCOUNTS_REST:AccountCellMap ) )
┃  ┊  subst: ...
┃  └─ 2 (leaf, target, terminal)
┃      k: #halt ~> CONTINUATION:K
┃      pc: PC_CELL_5d410f2a:Int
┃      callDepth: CALLDEPTH_CELL_5d410f2a:Int
┃      statusCode: STATUSCODE_FINAL:StatusCode
┃
┗━━┓ subst: .Subst
   ┃ constraint:
   ┃     ( maxUInt256 -Int KV1_b:Int ) <Int KV0_a:Int
   │
   ├─ 9
   │   k: #execute ~> CONTINUATION:K
   │   pc: 0
   │   callDepth: CALLDEPTH_CELL:Int
   │   statusCode: STATUSCODE:StatusCode
   │   src: test/Arithmeric.t.sol:3:20
   │   method: test%Arithmetic.add(uint256,uint256)
   │
   │  (999 steps)
   ├─ 7 (terminal)
   │   k: #halt ~> CONTINUATION:K
   │   pc: 605
   │   callDepth: CALLDEPTH_CELL:Int
   │   statusCode: EVMC_REVERT
   │   method: test%Arithmetic.add(uint256,uint256)
   │
   ┊  constraint:
   ┊      ( notBool <acctID>
  C_ARITHMETIC_ID:Int
</acctID> in_keys ( ACCOUNTS_REST:AccountCellMap ) )
   ┊  subst: ...
   └─ 2 (leaf, target, terminal)
       k: #halt ~> CONTINUATION:K
       pc: PC_CELL_5d410f2a:Int
       callDepth: CALLDEPTH_CELL_5d410f2a:Int
       statusCode: STATUSCODE_FINAL:StatusCode
```

As you can see, the summary covers two branches: one where the addition succeeds, and one where it overflows and reverts. During the execution of the test, this summary is being used as a high-priority semantic rule that gets inserted into the rule base (see [K Tutorial](https://kframework.org/docs/user_manual/) for more details on how rules are applied). Whenever the `add` function is called, this summary rule is applied instead of symbolically executing the function's implementation, directly determining the state transformation that corresponds to the function execution. 

If you look at the KCFG of the `testAdd` function, you will see that it does not contain nodes representing the execution of the `add` function, such as executing the `CALL` instruction, etc., but rather the target node(s) representing the result of this function execution:
```bash 
kontrol show 'ArithmeticTest.testAdd('
```

### Reducing Branching with Assumptions

The `testAdd` function does not include any assumptions on the values of `a` and `b`, so the summary is being used for both branches--including the overflow branch that ends in a `REVERT` instruction, making the proof fail. 

You can reduce the branching by adding assumptions to prevent overflow--a common solution is to add assumptions (`vm.assume()`) constraining the values of input parameters, as we did in the following test:
```solidity
function testAddWithAssumptions(uint256 a, uint256 b) public {
    vm.assume(a <= 2 ** 128 - 1);
    vm.assume(b <= 2 ** 128 - 1);
}
```
Here, the summary is being used for the branch where the addition succeeds, and the overflow branch is not covered by the summary, which can be checked using the `kontrol show` command too. Similarly, we have added a test that uses the `addAndIncrement` function, which will also use the summary for the branch where the addition succeeds, and the overflow branch is not covered by the summary. Both of these proofs should pass.

You can visualize the resulting KCFG by running:
```
kontrol show ArithmeticTest.testAddWithAssumptions
```

### Reducing Branching with NatSpec Preconditions

Alternatively, Kontrol supports NatSpec preconditions directly in Solidity code on the function being CSE'd using the `@custom:kontrol-precondition` annotation, as cheatcodes such as `vm.assume()` are not available outside of `Test` contracts during CSE:

```solidity
contract Arithmetic {
    /// @custom:kontrol-precondition a <= 2**128 - 1,
    /// @custom:kontrol-precondition b <= 2**128 - 1,
    function add(uint256 a, uint256 b) external pure returns (uint256) {
        return a + b;
    }
}
```

{% hint style="warning" %}
**Experimental Feature**: NatSpec preconditions are currently experimental and will be available in the next release of Kontrol (v1.0.188). While most popular Solidity terms and variable types are supported, some advanced features may not be available. If you need support for specific Solidity constructs, please reach out to the community or consider contributing to the project. Note that NatSpec preconditions with unsupported constructs will be ignored.
{% endhint %}


In the code snippet above, we are making similar assumptions as in the previous test, but this time they will be applied during CSE, eliminating the overflow branches and making the summary itself more efficient. To try it out, run `kontrol build` that will re-execute `forge build` to recompile the contracts with the new NatSpec comments. Then, run:
```bash
kontrol prove --match-test ArithmeticTest.testAdd --cse --reinit --verbose
```

Now, during the execution of `add`, you will see the logs indicating that the NatSpec preconditions are being detected and applied:
```
INFO 2025-09-17 15:38:37,460 kontrol.natspec - Adding NatSpec precondition: { true #Equals KV0_a <=Int ( 2 ^Int 128 -Int 1 ) }
INFO 2025-09-17 15:38:37,467 kontrol.natspec - Adding NatSpec precondition: { true #Equals KV1_b <=Int ( 2 ^Int 128 -Int 1 ) }
```

These preconditions will be included in the path constraints of the summary, and only branches where the preconditions are satisfied will be covered by it. You will notice that path constraints in the nodes of the KCFG nodes include the preconditions:
```
#And ( { true #Equals KV0_a:Int <=Int maxUInt128 }
#And ( { true #Equals KV1_b:Int <=Int maxUInt128 }
```
and the KCFG has a single branch:
```
┌─ 1 (root, init)
│   k: #execute ~> CONTINUATION:K
│   pc: 0
│   callDepth: CALLDEPTH_CELL:Int
│   statusCode: STATUSCODE:StatusCode
│   src: test/Arithmeric.t.sol:3:22
│   method: test%Arithmetic.add(uint256,uint256)
│
│  (1279 steps)
├─ 4 (terminal)
│   k: #halt ~> CONTINUATION:K
│   pc: 154
│   callDepth: CALLDEPTH_CELL:Int
│   statusCode: EVMC_SUCCESS
│   src: test/Arithmeric.t.sol:6:8
│   method: test%Arithmetic.add(uint256,uint256)
│
┊  constraint:
┊      ( notBool <acctID>
  C_ARITHMETIC_ID:Int
</acctID> in_keys ( ACCOUNTS_REST:AccountCellMap ) )
┊  subst: ...
└─ 2 (leaf, target, terminal)
    k: #halt ~> CONTINUATION:K
    pc: PC_CELL_5d410f2a:Int
    callDepth: CALLDEPTH_CELL_5d410f2a:Int
    statusCode: STATUSCODE_FINAL:StatusCode
```
In a similar way, you can apply NatSpec preconditions to the test function itself.


The current precondition support allows you to express constraints on function parameters, storage variables, and global variables that are automatically applied during symbolic execution. The following are the currently supported constructs:
- **Operators**: All standard Solidity operators (arithmetic, comparison, boolean, bitwise)
- **Variables**: Function parameters, storage variables, globally-accessible variables (msg.sender, block.timestamp, etc.)
- **Literals**: Integers (decimal/hex), scientific notation, unit suffixes (ether, gwei, days, etc.)
- **Expressions**: Binary operations, unary operations, member access for globals

Some Solidity constructs are not yet supported, including array/mapping access, nested member (e.g., struct) access limited to global variables, storage slot offsets not applied (for packed storage slots).

## Node Merging

{% hint style="warning" %}
**Experimental Feature**: Node merging is currently experimental, will be available in the next release of Kontrol (v1.0.188), and the merging heuristic we are using may not cover all scenarios. Performance improvements and behavior may vary depending on the specific code structure and branching patterns.
{% endhint %}

Node merging is another optimization technique that reduces proof complexity by combining similar execution paths. This is particularly useful when multiple branches converge to the same final state.

### How Node Merging Works

When Kontrol encounters multiple execution paths that lead to identical or very similar states, it can merge these paths into a single node, reducing the overall complexity of the proof.

More specifically, node merging reduces the branching factor of proofs by "pushing splits down" through the KCFG. The process works by:
1. **Identifying mergeable nodes**: Using a semantics-specific heuristic to determine if two nodes can be merged
2. **Computing merged nodes**: Creating a new node `B ∨ C` that represents the union of two nodes, with fresh variables where configurations differ
3. **Conditional instantiation**: The merged node contains conditions that instantiate it to the original nodes based on path conditions
4. **Restructuring the graph**: Removing the original nodes and inserting the merged node with appropriate edges

This technique is particularly effective when splits can be pushed all the way down to the target node of the proof, significantly reducing the overall branching factor. Node merging is especially powerful during CSE, where the same summary is being reused across multiple execution paths, compounding the reduction in branching factor and making verification more efficient.

### Example: Conditional Logic with Convergence

The `addAndIncrement` function in our arithmetic contract demonstrates node merging:

```solidity
function addAndIncrement(uint256 a, uint256 b, bool shouldIncrement) external pure returns (uint256) {
    uint256 result = a + b;
    
    if (shouldIncrement) {
        result = result + 1; // Increment by 1
    }
    
    // All paths converge here for final validation
    assert(result >= a + b);
    return result;
}
```

In this example, the conditional branch (incrementing or not) eventually converges to the same final validation state, making it a candidate for node merging. The different values of the `shouldIncrement` flag create execution paths that all end up at the same assertion and return statement.

Node merging is currently available during the minimization of a proof that has been generated with `kontrol prove`. After running a proof, you can minimize it with node merging enabled using the `kontrol minimize-proof` command by providing the proof name and the `--merge` flag:

```bash
# Minimizing the proof with node merging
kontrol minimize-proof ArithmeticTest.testAddAndIncrement --merge
```

Now, when you run `kontrol show ArithmeticTest.testAddAndIncrement` to visualize the proof again, you should see new merged edge leading to the target nodes, constrained by the path conditions of the original nodes, indicating that the previously distinct execution paths have been merged. If you run

```bash
kontrol show ArithmeticTest.testAddAndIncrement
```

you will see the following nodes and edges:
```
┌─ 1 (root, split, init)
│   k: #execute ~> CONTINUATION:K
│   pc: 0
│   callDepth: CALLDEPTH_CELL:Int
│   statusCode: STATUSCODE:StatusCode
│   src: test/Arithmeric.t.sol:3:20
│   method: test%Arithmetic.addAndIncrement(uint256,uint256,bool)
┃
┃ (branch)
┣━━┓ subst: .Subst
┃  ┃ constraint:
┃  ┃     ( ( maxUInt256 -Int KV1_b:Int ) <Int KV0_a:Int orBool ( ( notBool KV2_shouldIncrement:Int ==Int 0 ) andBool ( 115792089237316195423570985008687907853269984665640564039457584007913129639934 <Int ( KV0_a:Int +Int KV1_b:Int ) andBool KV0_a:Int <=Int ( maxUInt256 -Int KV1_b:Int ) ) ) )
┃  │
┃  ├─ 29
┃  │   k: #execute ~> CONTINUATION:K
┃  │   pc: 0
┃  │   callDepth: CALLDEPTH_CELL:Int
┃  │   statusCode: STATUSCODE:StatusCode
┃  │   src: test/Arithmeric.t.sol:3:20
┃  │   method: test%Arithmetic.addAndIncrement(uint256,uint256,bool)
┃  │
┃  │  (1228|1507 steps)
┃  ├─ 30 (split)
┃  │   k: #halt ~> CONTINUATION:K
┃  │   pc: 605
┃  │   callDepth: CALLDEPTH_CELL:Int
┃  │   statusCode: EVMC_REVERT
┃  │   method: test%Arithmetic.addAndIncrement(uint256,uint256,bool)
...
```

Here, you may notice that several branches have been merged into a single merged edge, guarded by the constraints representing the disjunction of the constraints of the original nodes, and marked with the `(1228|1507 steps)` label, indicating the number of steps taken in the original KCFG before the merge transformation.

Node merging is especially useful when dealing with high branching factors, particularly during CSE, where the branching factor is multiplied by the number of external calls, while node merging can provide arbitrary branch reduction, trimming the execution paths down to a manageable number.

We plan to automatically apply node merging during CSE summary generation in future releases, making this optimization more seamless, and further investigate the merging heuristic to cover more scenarios.

## Benefits of CSE and Node Merging

### Performance Improvements
- **Reduced path explosion**: External functions are analyzed once and summarized, with node merging further reducing the branching factor
- **Faster verification**: Reusing summaries eliminates redundant computation
- **Scalable to complex contracts**: Works well with deeply nested function calls

### Practical Advantages
- **Reusable summaries**: Once generated, summaries can be used across multiple tests and even projects, or serve as verification artifacts that can be shared with the community
- **Modular verification**: Each function is verified in isolation with abstract inputs
- **Better resource utilization**: Less memory and CPU usage for complex verification tasks

### Benefits of NatSpec Preconditions

1. **Declarative constraints**: Express preconditions directly in the code where they're most relevant, especially on functions being summarized
2. **Automatic application**: An alternative to manually adding `vm.assume()` calls in tests, which are not available during CSE
3. **Better documentation**: Preconditions serve as executable documentation
4. **Consistent verification**: Same constraints applied across all test scenarios
5. **Performance improvement**: Reduces the search space for symbolic execution

## Best Practices

### For CSE and Node Merging
1. **Design modular functions**: Break complex logic into smaller, focused functions
2. **Structure for convergence**: Design code so that similar execution paths naturally converge
3. **Place assertions at convergence points**: Verify merged states with meaningful assertions

### For NatSpec Preconditions
1. **Express clear constraints**: Write preconditions that clearly define valid input ranges
2. **Use meaningful expressions**: Make preconditions readable and self-documenting
3. **Test edge cases**: Verify that preconditions cover all necessary scenarios
4. **Balance expressiveness vs performance**: More specific preconditions improve performance but reduce the search space
5. **Consider overflow protection**: Include bounds checking for arithmetic operations which are a common cause of reverts
