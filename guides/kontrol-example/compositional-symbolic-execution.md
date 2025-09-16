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

Throughout this guide, we'll use a simple arithmetic contract to demonstrate CSE, node merging, and NatSpec preconditions:

```solidity
// src/Arithmetic.sol
pragma solidity ^0.8.13;

contract Arithmetic {
    function add(uint256 a, uint256 b) external pure returns (uint256) {
        return a + b;
    }
    
    function multiply(uint256 a, uint256 b) external pure returns (uint256) {
        return a * b;
    }
    
    function calculateWithDiscount(uint256 amount, bool isVIP, bool isLarge) external pure returns (uint256) {
        uint256 result = amount;
        
        // Multiple conditional branches that can converge
        if (isVIP) {
            result = result * 9 / 10; // 10% discount for VIP
        }
        
        if (isLarge && amount > 1000) {
            result = result * 9 / 10; // Additional 10% discount for large amounts
        }
        
        // All paths converge here for final validation
        assert(result <= amount);
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
        // Assume reasonable constraints to prevent overflow
        vm.assume(a <= type(uint256).max / 2);
        vm.assume(b <= type(uint256).max / 2);
        
        uint256 result = arithmetic.add(a, b);
        assert(result == a + b);
    }
    
    function testMultiply(uint256 a, uint256 b) public {
        // Assume reasonable constraints to prevent overflow
        vm.assume(a <= type(uint256).max / 2);
        vm.assume(b <= type(uint256).max / 2);
        
        uint256 result = arithmetic.multiply(a, b);
        assert(result == a * b);
    }
    
    function testCalculateWithDiscount(uint256 amount, bool isVIP, bool isLarge) public {
        vm.assume(amount > 0);
        vm.assume(amount <= type(uint256).max / 100); // Prevent overflow
        
        uint256 result = arithmetic.calculateWithDiscount(amount, isVIP, isLarge);
        assert(result <= amount);
    }
}
```

## Running CSE

To use CSE, simply add the `--cse` flag to your prove command:

```bash
# Build the project
kontrol build

# Run with CSE enabled
kontrol prove --match-test ArithmeticTest.testAdd --cse
```

When CSE is enabled, Kontrol will:
1. First analyze `add` in isolation to generate a summary
2. Then execute the test using the `add` summary

This eliminates the need to re-execute `add` for every path in the test, improving performance.

## Benefits of CSE

### Performance Improvements
- **Reduced path explosion**: External functions are analyzed once and summarized
- **Faster verification**: Reusing summaries eliminates redundant computation
- **Scalable to complex contracts**: Works well with deeply nested function calls

### Practical Advantages
- **Reusable summaries**: Once generated, summaries can be used across multiple tests and even projects, or serve as verification artifacts that can be shared with the community
- **Modular verification**: Each function is verified in isolation with abstract inputs
- **Better resource utilization**: Less memory and CPU usage for complex verification tasks

## NatSpec Preconditions

{% hint style="warning" %}
**Experimental Feature**: NatSpec preconditions are currently experimental. While most popular Solidity terms and variable types are supported, some advanced features may not be available. If you need support for specific Solidity constructs, please reach out to the community or consider contributing to the project.
{% endhint %}

Kontrol supports special NatSpec comments to specify preconditions directly in your Solidity code using the `@custom:kontrol-precondition` annotation. This feature allows you to express constraints on function parameters, storage variables, and global variables that are automatically applied during symbolic execution.

### Supported Annotations

- `@custom:kontrol-precondition`: Specifies preconditions as Solidity expressions
- `@custom:kontrol-array-length-equals`: Specifies the number of elements in an array
- `@custom:kontrol-bytes-length-equals`: Specifies the length of bytes or byte arrays

### Example: Using Preconditions with Arithmetic

```solidity
contract Arithmetic {
    /// @custom:kontrol-precondition a > 0, b > 0, a + b <= type(uint256).max
    function add(uint256 a, uint256 b) external pure returns (uint256) {
        return a + b;
    }
    
    /// @custom:kontrol-precondition a > 0, b > 0, a * b <= type(uint256).max
    function multiply(uint256 a, uint256 b) external pure returns (uint256) {
        return a * b;
    }
    
    /// @custom:kontrol-precondition amount > 0, amount <= type(uint256).max / 100
    function calculateWithDiscount(uint256 amount, bool isVIP, bool isLarge) external pure returns (uint256) {
        uint256 result = amount;
        
        if (isVIP) {
            result = result * 9 / 10; // 10% discount for VIP
        }
        
        if (isLarge && amount > 1000) {
            result = result * 9 / 10; // Additional 10% discount for large amounts
        }
        
        assert(result <= amount);
        return result;
    }
}
```

### Supported Features

The precondition parser supports:

- **Operators**: All standard Solidity operators (arithmetic, comparison, boolean, bitwise)
- **Variables**: Function parameters, storage variables, globally-accessible variables (msg.sender, block.timestamp, etc.)
- **Literals**: Integers (decimal/hex), scientific notation, unit suffixes (ether, gwei, days, etc.)
- **Expressions**: Binary operations, unary operations, member access for globals

### Example: Complex Preconditions

```solidity
contract AdvancedContract {
    uint256 public constant MAX_VALUE = 1000;
    address public owner;
    
    /// @custom:kontrol-precondition value > 0, value <= MAX_VALUE, msg.sender == owner, block.timestamp > 0
    function updateValue(uint256 value) external {
        // Function implementation
    }
    
    /// @custom:kontrol-precondition a != 0, b != 0, a * b <= type(uint256).max
    function multiply(uint256 a, uint256 b) external pure returns (uint256) {
        return a * b;
    }
}
```

### Benefits of NatSpec Preconditions

1. **Declarative constraints**: Express preconditions directly in the code where they're most relevant, especially on functions being summarized
2. **Automatic application**: An alternative to manually adding `vm.assume()` calls in tests, which are not available during CSE
3. **Better documentation**: Preconditions serve as executable documentation
4. **Consistent verification**: Same constraints applied across all test scenarios
5. **Performance improvement**: Reduces the search space for symbolic execution

### Current Limitations

- Some Solidity constructs are not yet supported, including array/mapping access, nested member (e.g., struct) access limited to global variables, storage slot offsets not applied (for packed storage slots).

## Node Merging

{% hint style="warning" %}
**Experimental Feature**: Node merging is currently experimental and may not work optimally in all scenarios. Performance improvements and behavior may vary depending on the specific code structure and branching patterns.
{% endhint %}

Node merging is another optimization technique that reduces proof complexity by combining similar execution paths. This is particularly useful when multiple branches converge to the same final state.

### How Node Merging Works

When Kontrol encounters multiple execution paths that lead to identical or very similar states, it can merge these paths into a single node, reducing the overall complexity of the proof.

More specifically, node merging reduces the branching factor of proofs by "pushing splits down" through the KCFG (K Control Flow Graph). The process works by:
1. **Identifying mergeable nodes**: Using a semantics-specific heuristic to determine if two nodes can be merged
2. **Computing merged nodes**: Creating a new node `B ∨ C` that represents the union of two nodes, with fresh variables where configurations differ
3. **Conditional instantiation**: The merged node contains conditions that instantiate it to the original nodes based on path conditions
4. **Restructuring the graph**: Removing the original nodes and inserting the merged node with appropriate edges

This technique is particularly effective when splits can be pushed all the way down to the target node of the proof, significantly reducing the overall branching factor. Node merging is especially powerful during CSE, where the same summary is being reused across multiple execution paths, compounding the reduction in branching factor and making verification more efficient.

### Example: Conditional Logic with Convergence

The `calculateWithDiscount` function in our arithmetic contract demonstrates node merging:

```solidity
function calculateWithDiscount(uint256 amount, bool isVIP, bool isLarge) external pure returns (uint256) {
    uint256 result = amount;
    
    // Multiple conditional branches that can converge
    if (isVIP) {
        result = result * 9 / 10; // 10% discount for VIP
    }
    
    if (isLarge && amount > 1000) {
        result = result * 9 / 10; // Additional 10% discount for large amounts
    }
    
    // All paths converge here for final validation
    assert(result <= amount);
    return result;
}
```

In this example, multiple conditional branches (VIP status, large amount discounts) eventually converge to the same final validation state, making it an excellent candidate for node merging. The different combinations of `isVIP` and `isLarge` flags create multiple execution paths that all end up at the same assertion and return statement.

### Using Node Merging

Node merging is currently available during the minimization of a proof that has been generated with `kontrol prove`. After running a proof, you can minimize it with node merging enabled using the `kontrol minimize-proof` command by providing the proof name and the `--merge` flag:

```bash
# First, run the proof
kontrol prove --match-test ArithmeticTest.testCalculateWithDiscount

# Then minimize the proof with node merging
kontrol minimize-proof ArithmeticTest.testCalculateWithDiscount --merge
```

Now, when you run `kontrol show ArithmeticTest.testCalculateWithDiscount` to visualize the proof again, you should see a new merged edge leading to the merged node, indicating that the previously distinct execution paths have been merged:

We plan to automatically apply node merging during CSE summary generation in future releases, making this optimization more seamless.
```bash
kontrol show ArithmeticTest.testCalculateWithDiscount
```

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
