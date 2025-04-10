---
description: Tips and tricks for debugging failing proofs in Kontrol
---

# Debugging Failing Proofs

When a proof fails in Kontrol, there are several strategies you can use to debug and potentially resolve the issue. Here are some common approaches:

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

### Using K Framework Assumptions
For more complex cases, you can use K Framework assumptions directly:

```k
rule <k> ... </k>
     <storage> STORAGE </storage>
     requires { true #Equals notBool #lookup(STORAGE, 2) <=Int chop(#lookup(STORAGE, 2) +Int 1) }
```

## Other Debugging Tips

1. **Simplify the Proof**
   - Break down complex proofs into smaller, more manageable parts
   - Verify individual components before combining them
   - Use concrete values for some variables to reduce complexity

2. **Check Path Conditions**
   - Use `kontrol view-kcfg` to inspect the control flow graph
   - Look for unexpected branching conditions
   - Verify that all paths are being explored as expected

3. **Reduce Symbolic Variables**
   - Use concrete values where possible
   - Limit the range of symbolic variables
   - Add more constraints to reduce the search space

4. **Use Lemmas**
   - Create and prove lemmas for complex properties
   - Use lemmas to break down the proof into smaller steps
   - Leverage existing lemmas from the KEVM library

5. **Check Storage Layout**
   - Verify that storage slots are being accessed correctly
   - Ensure that storage updates are happening in the expected order
   - Check for potential storage collisions

Debugging formal verification proofs often requires a combination of these approaches. Start with the simplest solution and gradually move to more complex ones if needed. 