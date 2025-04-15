---
description: Learn about Kontrol's cheatcodes for testing and verification
---

# Kontrol Cheatcodes

Kontrol implements a subset of Foundry's cheatcodes, focusing on those most relevant for formal verification. These cheatcodes enable powerful testing and verification capabilities, with some additional functionality specific to symbolic execution and formal verification.

For a comprehensive technical reference of all cheatcodes and their implementations, see the [Kontrol cheatcodes source](https://github.com/runtimeverification/kontrol/blob/master/src/kontrol/kdist/cheatcodes.md).

## Configuration Overview

Kontrol's cheatcodes are organized into several key configurations:

### Prank Configuration
Used for simulating calls from different addresses:
- `<prevCaller>`: Current address of the contract initiating the prank
- `<prevOrigin>`: Current `tx.origin` value
- `<newCaller>` and `<newOrigin>`: Addresses to assign to `msg.sender` and `tx.origin`
- `<active>`: Indicates if a prank is currently active
- `<depth>`: Current call depth at which the prank was invoked
- `<singleCall>`: Determines if the prank stops after next call or requires `stopPrank`

### Expected Revert Configuration
Used for testing expected reverts:
- `<isRevertExpected>`: Flags if the next call should revert
- `<expectedDepth>`: Depth at which the call should revert
- `<expectedReason>`: Expected revert message as Bytes

### Expected Opcode Configuration
Used for verifying specific opcode calls:
- `<isOpcodeExpected>`: Flags if a call opcode is expected
- `<expectedAddress>`: Expected caller address
- `<expectedValue>`: Expected `msg.value`
- `<expectedData>`: Expected `calldata`
- `<opcodeType>`: Type of `CALL*` opcode expected

### Expected Emit Configuration
Used for event verification:
- `<recordEvent>`: Flags if next event should be recorded
- `<isEventExpected>`: Flags if an event should match previously recorded
- `<checkedTopics>`: List of bools indicating which topics to check
- `<checkedData>`: Flag for checking data field
- `<expectedEventAddress>`: Address of expected event emitter

### Whitelist Configuration
Used for controlling function calls and storage access during execution:
- `<isCallWhitelistActive>`: Enables whitelist mode for calls
- `<isStorageWhitelistActive>`: Enables whitelist mode for storage
- `<addressList>`: List of whitelisted addresses that are allowed to be called
- `<storageSlotList>`: List of whitelisted storage slots that are allowed to be modified

### Mock Calls Configuration
Used for mocking contract calls:
- `<mockCall>`: Collection of active mock calls per address
  - `<mockAddress>`: Address with active mock calls
  - `<mockValues>`: Map of calldata to returndata

## Available Cheatcodes

### Address Manipulation
- `prank(address)`: Change `msg.sender` for next call
- `prank(address,address)`: Change both `msg.sender` and `tx.origin`

### Storage and State
- `symbolicStorage(address)`: Make storage symbolic
- `symbolicStorage(address,string)`: Make storage symbolic with custom name
- `setArbitraryStorage(address)`: Allow arbitrary storage modifications
- `copyStorage(address,address)`: Copy storage between contracts

### Symbolic Values
- `freshUInt(uint8)`: Generate fresh symbolic uint
- `freshUInt(uint8,string)`: Generate fresh symbolic uint with custom name
- `freshBool()`: Generate fresh symbolic bool
- `freshBool(string)`: Generate fresh symbolic bool with custom name
- `freshBytes(uint256)`: Generate fresh symbolic bytes
- `freshBytes(uint256,string)`: Generate fresh symbolic bytes with custom name
- `freshAddress()`: Generate fresh symbolic address
- `freshAddress(string)`: Generate fresh symbolic address with custom name

### Random Values
- `randomUint()`: Generate random uint
- `randomUint(uint256)`: Generate random uint with bound
- `randomUint(uint256,uint256)`: Generate random uint in range
- `randomBool()`: Generate random bool
- `randomBytes(uint256)`: Generate random bytes
- `randomBytes4()`: Generate random bytes4
- `randomBytes8()`: Generate random bytes8
- `randomAddress()`: Generate random address

### Gas Manipulation
- `infiniteGas()`: Set infinite gas for next call
- `setGas(uint256)`: Set specific gas amount

### Mock Calls
- `mockCall(address,bytes,bytes)`: Mock a call with specific calldata and returndata
- `mockFunction(address,address,bytes)`: Mock a specific function call

### Branch Management
- `forgetBranch(uint256,uint8,uint256)`: Forget a specific branch in the proof

## Usage Examples

### Symbolic Testing
```solidity
function testSymbolicValues() public {
    uint256 x = vm.freshUInt(256, "x");
    bool b = vm.freshBool("b");
    address a = vm.freshAddress("a");
    
    // Your test logic here
}
```

### Mock Calls
```solidity
function testMockCalls() public {
    bytes memory mockCalldata = abi.encodeWithSignature("balanceOf(address)", address(this));
    bytes memory mockReturndata = abi.encode(1000);
    vm.mockCall(tokenAddress, mockCalldata, mockReturndata);
    
    // Your test logic here
}
```

### Storage Manipulation
```solidity
function testStorage() public {
    vm.symbolicStorage(contractAddress);
    // Your storage manipulation and verification here
}
```

### Access Control Testing
```solidity
function testAccessControl() public {
    address attacker = vm.freshAddress("attacker");
    vm.prank(attacker);
    // Attempt restricted operation
    vm.expectRevert("Unauthorized");
    restrictedContract.restrictedFunction();
}
```

### Symbolic Storage Testing
```solidity
function testSymbolicStorage() public {
    // Make contract storage symbolic
    vm.symbolicStorage(address(token));
    
    // Test with arbitrary storage values
    uint256 balance = token.balanceOf(address(this));
    require(balance > 0, "Balance should be positive");
}
```

### Complex Mock Setup
```solidity
function testComplexMock() public {
    // Setup symbolic values
    address user = vm.freshAddress("user");
    uint256 amount = vm.freshUInt(256, "amount");
    
    // Mock token transfer
    bytes memory transferCalldata = abi.encodeWithSignature(
        "transfer(address,uint256)", 
        user, 
        amount
    );
    bytes memory transferResult = abi.encode(true);
    vm.mockCall(tokenAddress, transferCalldata, transferResult);
    
    // Test the interaction
    bool success = token.transfer(user, amount);
    assert(success);
}
```

## Best Practices

1. Use symbolic values for inputs that should be explored exhaustively
2. Mock external calls to control their behavior during verification and isolate the contract under test
3. Use `prank` to test access control and permission checks
4. Leverage symbolic storage for comprehensive storage and contract behavior analysis
5. Name your symbolic variables meaningfully for better debugging
6. Combine symbolic and concrete values when appropriate

## Limitations

Some Foundry cheatcodes are not yet implemented in Kontrol, including:
- Environment variable manipulation (`setEnv`, `envBool`, etc.)
- File operations (`readFile`, `writeFile`, etc.)
- Fork management (`createFork`, `selectFork`, etc.)
- FFI operations
- String conversion utilities 