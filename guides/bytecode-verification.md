# Bytecode Verification

## Overview

Kontrol supports verifying contracts using only their bytecode through the `vm.etch` cheatcode. This is particularly useful when:
- You want to verify interactions with contracts that only have bytecode available on-chain
- You need to verify contracts where the source code is not available
- You want to simulate mainnet forking scenarios

## Basic Setup

To verify a contract using its bytecode, you'll need to:
1. Get the contract's bytecode (either from on-chain or other sources)
2. Use `vm.etch` to deploy the bytecode at a specific address
3. Create an interface for the contract
4. Write and run your verification tests

Here's a basic example:

```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.13;

import {Test} from "forge-std/Test.sol";
import {KontrolCheats} from "kontrol-cheatcodes/KontrolCheats.sol";

// Define the interface for the contract you want to verify
interface IUnknownContract {
    function someFunction(uint256) external returns (bool);
}

contract BytecodeVerificationTest is Test, KontrolCheats {
    IUnknownContract public unknownContract;
    address constant UNKNOWN_CONTRACT_ADDRESS = address(0x1234);

    function setUp() public {
        // Get the bytecode (this would typically come from on-chain or other sources)
        bytes memory bytecode = hex"608060405234801561001057600080fd5b506004361061002b5760003560e01c8063..."; // truncated for example

        // Deploy the bytecode at the specified address
        vm.etch(UNKNOWN_CONTRACT_ADDRESS, bytecode);
        
        // Initialize the interface
        unknownContract = IUnknownContract(UNKNOWN_CONTRACT_ADDRESS);
    }

    function testFuzz_SomeFunction(uint256 x) public {
        // Make the storage symbolic
        vm.setArbitraryStorage(UNKNOWN_CONTRACT_ADDRESS);
        
        // Call the function and verify properties
        bool result = unknownContract.someFunction(x);
        // Add your verification properties here
    }
}
```

## Getting Bytecode

There are several ways to get contract bytecode:

1. **From Etherscan**:
   - Navigate to the contract's page
   - Click on "Contract" tab
   - Click "Code" to see the bytecode

2. **Using Foundry's cast**:
   ```bash
   cast code <address> --rpc-url <your-rpc-url>
   ```

3. **Using web3.js/ethers.js**:
   ```javascript
   const bytecode = await provider.getCode(address);
   ```

## Best Practices

1. **Storage Handling**:
   - Use `vm.setArbitraryStorage` to make the contract's storage symbolic
   - This allows verification across all possible storage states

2. **Interface Definition**:
   - Define complete interfaces for all functions you want to verify
   - Include all necessary function signatures and events

3. **Address Management**:
   - Use consistent addresses across tests
   - Consider using `vm.createSelectFork` for mainnet simulations

4. **Gas Considerations**:
   - Bytecode verification can be more gas-intensive
   - Consider using `--use-gas` flag for more accurate gas analysis


## Additional Resources

- [Foundry's vm.etch documentation](https://book.getfoundry.sh/cheatcodes/etch)
- [Kontrol Cheatcodes Guide](../guides/cheatcodes.md)
- [Debugging Failing Proofs](../tips/debugging-failing-proofs.md) 