---
description: How to work with dynamically sized types in Kontrol using NatSpec annotations
---

# Working with Dynamically Sized Inputs

Kontrol provides powerful capabilities for working with dynamically sized types like arrays and byte arrays through NatSpec annotations. This allows you to specify constraints on symbolic variables that would otherwise have unbounded symbolic length, making proofs fail on compiler-inserted checks for calldata well-formedness.

## Overview

When working with symbolic execution, dynamically sized types like `bytes[]`, `uint256[]`, or `bytes` can have symbolic lengths, which can make verification complex or even impossible. Kontrol addresses this through custom NatSpec annotations that allow you to specify concrete sizes for these types.

## NatSpec Annotations

Kontrol supports two main NatSpec annotations for controlling dynamically sized types:

- `@custom:kontrol-array-length-equals`: Specifies the number of elements in an array
- `@custom:kontrol-bytes-length-equals`: Specifies the length of bytes or byte arrays

### Basic Example

```solidity
/// @custom:kontrol-array-length-equals ba: 2,
/// @custom:kontrol-bytes-length-equals ba: 600,
function test_complex_type(bytes[] calldata ba) public {
    require(ba.length == 2, "DynamicTypes: invalid length for bytes[]");
    assert(ba[1].length == 600);
}
```

In this example:
- The `ba` array will have exactly 2 elements
- Each element in `ba` will be exactly 600 bytes long
- The verification will only explore cases where these constraints are satisfied

## Real-World Example: Bridge Verification

Inspired by [Optimism's proofs](https://github.com/ethereum-optimism/optimism/blob/c38ce096def52e4acdaecb7ffd2d396f464693dd/packages/contracts-bedrock/test/kontrol/proofs/OptimismPortal.k.sol), here's a simplified example of how you might verify a bridge contract that processes cross-chain messages:

```solidity
/// @custom:kontrol-array-length-equals proof: 3,
/// @custom:kontrol-bytes-length-equals proof: 32,
function test_process_message(
    bytes calldata message,
    bytes[] calldata proof,
    uint256 blockNumber
) public {
    require(proof.length == 3, "Must have 3 proof elements");
    require(proof[0].length == 32, "Each proof element must be 32 bytes");
    
    // Simulate processing a cross-chain message with merkle proof
    bool success = bridgeContract.processMessage(message, proof, blockNumber);
    assert(success);
}
```

This test:
- Constrains `proof` to have exactly 3 elements (typical for a merkle proof)
- Each element in `proof` is exactly 32 bytes long (standard hash size)
- Tests the bridge message processing with these specific constraints

## Complex Types: Structs and Tuples

For more complex scenarios involving structs or tuples with dynamic types, Kontrol supports constraining the dynamic elements within these structures:

```solidity
struct ComplexType {
    uint256 id;
    bytes content;
}

/// @custom:kontrol-array-length-equals ctValues: 10,
/// @custom:kontrol-bytes-length-equals content: 10000,
function test_dynamic_struct_array(ComplexType[] calldata ctValues) public {
    require(ctValues.length == 10, "DynamicTypes: invalid length for ComplexType[]");
    assert(ctValues[8].content.length == 10000); // this check was failing
}
```

This example demonstrates:
- A struct `ComplexType` with a `uint256` id and dynamic `bytes` content
- Constraining the array `ctValues` to have exactly 10 elements
- Each `content` field within the structs is exactly 10,000 bytes long
- Verification that the 9th element (index 8) has the correct content length

## Usage Patterns

### Multiple Arrays

You can constrain multiple arrays in the same function:

```solidity
/// @custom:kontrol-array-length-equals addresses: 3,
/// @custom:kontrol-array-length-equals amounts: 3,
/// @custom:kontrol-bytes-length-equals addresses: 20,
function test_batch_transfer(
    address[] calldata addresses,
    uint256[] calldata amounts
) public {
    require(addresses.length == 3, "Must have 3 addresses");
    require(amounts.length == 3, "Must have 3 amounts");
    // Test logic here
}
```

## Best Practices

1. **Choose appropriate sizes**: Select sizes that are realistic for your use case but small enough for efficient verification
2. **Document constraints**: Use clear comments explaining why specific sizes were chosen
3. **Test edge cases**: Consider testing with different size combinations

## Limitations

- **Size constraints**: You must specify concrete sizes; symbolic lengths are not supported
- **Performance**: Larger sizes can significantly impact verification performance
- **Compatibility**: These annotations are Kontrol-specific and won't work with standard Foundry

## References

- [Optimism Kontrol Tests](https://github.com/ethereum-optimism/optimism/tree/c38ce096def52e4acdaecb7ffd2d396f464693dd/packages/contracts-bedrock/test/kontrol)
