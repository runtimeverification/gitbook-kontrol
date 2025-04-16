---
description: Learn about Kontrol's cheatcodes for testing and verification
---

# Kontrol Cheatcodes

Kontrol implements many Foundry's cheatcodes, focusing on those most relevant for formal verification. These cheatcodes enable powerful testing and verification capabilities, with some additional functionality specific to symbolic execution and formal verification.

For a comprehensive technical reference of all cheatcodes and their implementations, see the [Kontrol cheatcodes source](https://github.com/runtimeverification/kontrol/blob/master/src/kontrol/kdist/cheatcodes.md).

If you notice a missing cheatcode, feel free to open an [issue](https://github.com/runtimeverification/kontrol/issues) or [contribute](https://github.com/runtimeverification/kontrol/blob/master/CONTRIBUTING.md) directly to the project.


## Available Cheatcodes

### Foundry-Compatible Cheatcodes

#### Address Manipulation
- `prank(address)`: Change `msg.sender` for next call
- `prank(address,address)`: Change `msg.sender` and `tx.origin`
- `startPrank(address)`: Impersonate `msg.sender` until `stopPrank`
- `startPrank(address,address)`: Impersonate `msg.sender` and `tx.origin` until `stopPrank`
- `stopPrank()`: Stop impersonation

#### Account and Blockchain State
- `setArbitraryStorage(address)`: Make storage of an address symbolic
- `load(address,bytes32)`: Load storage slot
- `store(address,bytes32,bytes32)`: Set storage slot
- `copyStorage(address,address)`: Copy storage between contracts
- `deal(address,uint256)`: Set ETH/native token balance
- `etch(address,bytes)`: Set account code
- `getNonce(address)`: Get account nonce
- `setNonce(address,uint64)`: Set account nonce

#### Blockchain Environment
- `warp(uint256)`: Set block timestamp
- `roll(uint256)`: Set block number
- `fee(uint256)`: Set block base fee
- `chainId(uint256)`: Set chain ID
- `coinbase(address)`: Set block coinbase

#### Symbolic Execution Helpers
- `assume(bool)`: Add assumption for symbolic execution
- `randomUint()`: Generate a fresh symbolic uint
- `randomUint(uint256,uint256)`: Generate symbolic uint in range
- `randomBool()`: Generate symbolic bool
- `randomBytes(uint256)`: Generate symbolic bytes of a given size
- `randomAddress()`: Generate symbolic address
- `randomBytes4()`: Generate symbolic bytes4
- `randomBytes8()`: Generate symbolic bytes8

#### Address Utilities
- `label(address,string)`: Label address for debugging
- `addr(uint256)`: Compute address from private key

#### Revert and Event Expectation Assertions
- `expectRevert()`: Expect revert for next call
- `expectRevert(bytes4)`: Expect revert with specific selector
- `expectRevert(bytes)`: Expect revert with specific message
- `expectEmit(bool,bool,bool,bool)`: Expect event for next call
- `expectEmit(bool,bool,bool,bool,address)`: Expect event with specific emitter address

#### Cryptographic Utilities
- `sign(uint256,bytes32)`: Sign digest with private key

#### Mocking
- `mockCall(address,bytes,bytes)`: Mock call with calldata and returndata
- `mockFunction(address,address,bytes)`: Mock function by redirecting a call to a mock contract

### Kontrol-Specific Cheatcodes

{% hint style="info" %}
Using Kontrol-specific cheatcodes will break compatibility with `forge test`, as `forge` will not recognize these cheatcodes. Prefer Foundry-compatible cheatcodes where possible.
{% endhint %}

#### Symbolic Storage
- `symbolicStorage(address)`: Make storage of an address symbolic, alias for `setArbitraryStorage`
- `symbolicStorage(address,string)`: Make storage of an address symbolic with custom name

#### Symbolic Values (Custom-Named)
- `freshUInt(uint8)`: Generate symbolic uint of given size, alias for `randomUint(uint256)`
- `freshUInt(uint8,string)`: Generate symbolic uint of a given size with custom name
- `freshBool()`: Generate symbolic bool, alias for `randomBool()`
- `freshBool(string)`: Generate symbolic bool with custom name
- `freshBytes(uint256)`: Generate symbolic bytes of a given size, alias for `randomBytes(uint256)`
- `freshBytes(uint256,string)`: Generate symbolic bytes of a given size with custom name
- `freshAddress()`: Generate symbolic address, alias for `randomAddress()`
- `freshAddress(string)`: Generate symbolic address with custom name

#### Gas Manipulation
- `infiniteGas()`: Assume the execution can use infinite gas
- `setGas(uint256)`: Set execution gas to the given value

#### Branch Management
- `forgetBranch(uint256,uint8,uint256)`: Forget branch condition during verification

#### Storage Changes and Call Whitelisting
- `allowChangesToStorage(address,uint256)`: Allow storage changes only to a specific address and slot
- `allowCallsToAddress(address)`: Allow calls only to a specific address
- `allowCalls(address,bytes)`: Allow calls only to a specific address and with specific calldata

### Call Expectation Assertions
- `expectStaticCall(address, bytes)`: Expect a `STATICCALL` to an address with given calldata
- `expectDelegateCall(address, bytes)`: Expects a `DELEGATECALL` to an address with given calldata
- `expectRegularCall(address, uint256, bytes)`: Expect `CALL` to an address, with the given `msg.value` and calldata
- `expectCreate(address, uint256, bytes)`: Expect `CREATE` initiated by the specified address, with given `msg.value` and bytecode
- `expectCreate2(address, uint256, bytes)`: Expects `CREATE2` from the given address, sending specified `msg.value` and bytecode

## Usage Examples

### Symbolic Storage Testing
```solidity
function testSymbolicStorage(address to) public {
    // Make storage of `token` symbolic
    vm.setArbitraryStorage(address(token));
    // Retrieve an arbitrary storage value 
    uint256 balance = token.balanceOf(address(this));
    // Assume it is is positive
    vm.assume(balance > 0);
    // Perform some operations
    token.transfer(address(to), balance);
    // Check the result
    assert(token.balanceOf(address(this)) == 0);
}
```

### Access Control Testing
```solidity
function test_ccessControl(address caller) public {
    // Assume a symbolic caller is 
    // Call
    vm.prank(caller);
    // Attempt restricted operation
    vm.expectRevert(bytes4(keccak256("Unauthorized")));
    restrictedContract.restrictedFunction();
}
```

### Symbolic Testing
```solidity
function testSymbolicValues() public {
    // Generate symbolic values
    uint256 x = vm.randomUint();
    bool b = vm.randomBool();
    address a = vm.randomAddress();
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

### Complex Mock Setup
```solidity
function testComplexMock() public {
    // Setup symbolic values
    address user = vm.randomAddress();
    uint256 amount = vm.randomUint();
    
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
2. Use `prank` to test access control and permission checks
3. Leverage symbolic storage for comprehensive storage and contract behavior analysis
4. Name your symbolic variables meaningfully when debugging
5. Mock external calls to control their behavior during verification and isolate the contract under test
6. Combine symbolic and concrete values strategically when appropriate

## Limitations

Some Foundry cheatcodes are not yet implemented in Kontrol, including:
- Environment variable manipulation (`setEnv`, `envBool`, etc.)
- File operations (`readFile`, `writeFile`, etc.)
- Fork management (`createFork`, `selectFork`, etc.)
- FFI operations
- String conversion utilities 

## Configuration Overview

In the KEVM configuration, Kontrol's cheatcodes are organized into several key cells:

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