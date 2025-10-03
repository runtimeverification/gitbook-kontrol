---
description: Generating counterexamples for failing proofs in Kontrol
---

# Counterexample Generation

## Overview

Counterexample generation is a new experimental feature in Kontrol that helps developers understand why their proofs fail by providing concrete examples of inputs and execution paths that violate the properties being tested. It enables the users to easily reproduce the failing test case with Foundry and is essential for debugging and improving smart contract verification.

## What Are Counterexamples?

A counterexample in the context of formal verification is a concrete execution trace that demonstrates how a property can be violated. In Kontrol, counterexamples show input values that lead to property violations and the path conditions that should be satisfied for the failure to occur.

## How Counterexample Generation Works

Kontrol generates counterexamples by:

1. **Analyzing failed proofs**: When a proof fails, Kontrol identifies the specific execution path that led to the failure and generates the model that demonstrates the concrete values that symbolic variables should take for the failure to happen
2. **Extracting concrete values**: Symbolic variables are instantiated with concrete values that trigger the failure
3. **Constructing execution trace**: The complete execution path is reconstructed with concrete values in the form of a Foundry test case and is saved in a new file

## Basic Usage

### Step 1: Enable Counterexample Generation

Counterexample generation is enabled by providing the `--generate-counterexample` option to `kontrol prove`. When a proof fails, Kontrol automatically generates concrete counterexample test functions:

```bash
kontrol prove --function testMyProperty --generate-counterexample
```

### Step 2: Generated Counterexample Functions

When a proof fails, Kontrol generates a new Solidity test file with concrete counterexample functions. Here's the actual example from [PR #1084](https://github.com/runtimeverification/kontrol/pull/1084):

**Original Failing Test:**
```solidity
function test_storage_setup(
    uint256 totalSupply, 
    uint256 _totalSupply, 
    address _owner, 
    uint256 _currentUser_id, 
    address _currentUser_wallet, 
    bool _currentUser_isActive, 
    uint256 _anotherTotalSupply
) public {
    assert(totalSupply != 0);
    simpleStorageStorageSetup(address(simpleStorage), _totalSupply, _owner, _currentUser_id, _currentUser_wallet, _currentUser_isActive);
    assert(simpleStorage.totalSupply() == _anotherTotalSupply);
}
```

**Generated Counterexample Functions:**
```solidity
function test_storage_setup_ce0(
    uint256 totalSupply, 
    uint256 _totalSupply, 
    address _owner, 
    uint256 _currentUser_id, 
    address _currentUser_wallet, 
    bool _currentUser_isActive, 
    uint256 _anotherTotalSupply
) public {
    // Counterexample values from failed proof:
    totalSupply = 0;
    _totalSupply = 0;
    _owner = address(0);
    _currentUser_id = 0;
    _currentUser_wallet = address(0);
    _currentUser_isActive = false;
    _anotherTotalSupply = 0;

    assert(totalSupply != 0);
    simpleStorageStorageSetup(address(simpleStorage), _totalSupply, _owner, _currentUser_id, _currentUser_wallet, _currentUser_isActive);
    assert(simpleStorage.totalSupply() == _anotherTotalSupply);
}

function test_storage_setup_ce1(
    uint256 totalSupply, 
    uint256 _totalSupply, 
    address _owner, 
    uint256 _currentUser_id, 
    address _currentUser_wallet, 
    bool _currentUser_isActive, 
    uint256 _anotherTotalSupply
) public {
    // Counterexample values from failed proof:
    totalSupply = 1;
    _totalSupply = 1;
    _owner = address(0);
    _currentUser_id = 0;
    _currentUser_wallet = address(0);
    _currentUser_isActive = false;
    _anotherTotalSupply = 0;

    assert(totalSupply != 0);
    simpleStorageStorageSetup(address(simpleStorage), _totalSupply, _owner, _currentUser_id, _currentUser_wallet, _currentUser_isActive);
    assert(simpleStorage.totalSupply() == _anotherTotalSupply);
}
```

**Key Features:**
- **Multiple failing nodes**: Generates indexed counterexample functions (`_ce0`, `_ce1`) for different failure scenarios
- **Concrete values**: Automatically maps symbolic variables to concrete values that trigger the failure
- **Automatic file generation**: Creates `{OriginalTestContractName}CounterexampleTest.t.sol` in the same directory
- **Type conversion**: Properly handles Solidity type conversions between SMT model and Solidity types

## Generated Files

The counterexample generation creates a new test file with the naming pattern:
- **Original file**: `MyContractTest.t.sol`
- **Generated file**: `MyContractTestCounterexampleTest.t.sol`

The generated file is placed in the same directory as the original test file to maintain accurate import paths.

## Development Workflow

1. **Write property**: Define the property you want to verify
2. **Run initial proof**: `kontrol prove --function testProperty --generate-counterexample`
3. **Analyze counterexample**: If it fails, examine the generated counterexample
4. **Fix the issue**: Use the counterexample to identify and fix the problem
5. **Re-run proof**: Verify the fix resolves the counterexample
6. **Iterate**: Repeat until the proof passes

## Related Features

- [Structured Symbolic Storage Generation](./structured-symbolic-storage-generation.md) - Automated storage setup
- [Symbolic Storage](./advancing-proofs/symbolic-storage.md) - Manual storage management
- [Debugging Failing Proofs](../tips/debugging-failing-proofs.md) - General debugging guide

## References

- [Kontrol Cheatcodes Reference](../cheatsheets/kontrol-cheatsheet.md)
