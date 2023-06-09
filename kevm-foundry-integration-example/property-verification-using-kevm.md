# Property Verification using KEVM

First, we need to modify our test to support symbolic execution. To do this we must import a new Solidity library with cheat codes required for symbolic execution. To do so, create a new file `src/KEVMCheats.sol` and copy over the contents of [this contract](https://github.com/runtimeverification/foundry-demo/blob/master/src/utils/KEVMCheats.sol). \
\
These cheat codes allow us to generalize the storage of an Ethereum account by making it symbolic or to abstract out gas usage by making it infinite. This can remove branches where the execution might fail because you run out of gas. For now, we will make use of the `infiniteGas` cheat code. We will present more complex examples in a future section.&#x20;

Note: this second change will prevent us from running the tests with `forge`, as `forge` will not recognize these cheat codes.

Now, `Counter.t.sol` should look like this:

```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.13;

import "forge-std/Test.sol";
import "../src/Counter.sol";
import "../src/KEVMCheats.sol";

contract CounterTest is Test, KEVMCheats {
   Counter public counter;
   
   function setUp() public {
       counter = new Counter();
       counter.setNumber(0, false);
   }
   
   function testIncrement() public {
        counter.increment();
        assertEq(counter.number(), 1);
    }
    
   function testSetNumber(uint256 x, bool inLuck) public {
       kevm.infiniteGas();
       counter.setNumber(x, inLuck);
       assertEq(counter.number(), x);
   }
}
```

To use symbolic execution, ensure that the contracts are compiled, and the artifacts generated by the solc compiler are up to date. To do this, run `forge build` in any folder of the repository. It’s as simple as that!&#x20;

This command will invoke the Solidity compiler on your project's contracts and tests.

Next, build a **KEVM** semantic definition for your Foundry project. Our tool inspects the compiled artifacts generated by solc and produces **KEVM** helper modules containing information for each Solidity contract found. Each helper module contains productions and rules for ABI properties and bytecode.  If you are curious, you can view the `out/kompiled/foundry.k` file. To begin, run the command below:

```sh
kevm foundry-kompile
```

The process should take a minute and may emit some warnings, so don’t worry. Remember that during the development, you may need to rebuild the definition in various ways.&#x20;

For example: If you change the Solidity code, you must re-run `forge build`, then run the `foundry-kompile` command again with the `--regen` option.

For more information about `foundry-kompile` and available options, refer to [the **KEVM** documentation](https://github.com/runtimeverification/evm-semantics/blob/master/include/kframework/foundry.md) or run `kevm foundry-kompile --help`.

Once you have `kompiled` the definition, you can run tests symbolically:&#x20;

```
kevm foundry-prove --test CounterTest.testSetNumber
```

The `--test CounterTest.testSetNumber` flag is used to specify that only a single proof should be executed. This is useful when there are multiple tests in a test file. The proof should fail after running for 15 minutes.&#x20;

Note: The time it takes to run can vary depending on the machine. You can add --verbose to see what is happening if it appears to be stalling.

Next, we will cover how to investigate why a test failed.
