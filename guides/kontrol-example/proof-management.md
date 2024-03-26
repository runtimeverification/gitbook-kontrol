# Proof Management

In this section, we will take a closer look at **Kontrol**'s proof management and provide you with essential information to effectively navigate the platform.

## Viewing Proofs

To view the proofs that have been executed, you can use the `kontrol list` command. This will display a list of executed proofs. If a proof has been executed multiple times, you will see a version number (`:#`) after the function name. It is important to note the version number when specifying which proof you want to select, especially when viewing the **K Control Flow Graph** (**KCFG**) for a specific proof.&#x20;

The image below shows orange circles around the version numbers of the proofs obtained after running `kontrol list`.

<figure><img src="../../.gitbook/assets/Screenshot 2024-03-05 at 7.21.14 PM (1).png" alt=""><figcaption></figcaption></figure>

Another way to view the executed proofs is by checking the project directory. In the case of `kontrolexample`, you can find the executed proofs in `kontrolexample/out/proofs`.

<figure><img src="../../.gitbook/assets/Untitled 2.png" alt=""><figcaption></figcaption></figure>

If you don't see a proof displayed when running `kontrol list`, please check the directory. The command `kontrol list` only shows the latest proof run, and maybe not display previous ones. However, the previous proofs are still stored in the directory, even though they may not show up in the CLI. The following section will attempt to address some of these complexities.

## Making a Change and Rerunning a Proof

Building upon the example from the previous pages, we will now make another change to `Counter.sol` and create a new proof. Specifically, we are going to change the data type of `inLuck` from `bool` to `bytes32`. After this change, `Counter.sol` will appear as follows:

```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.13;

error CoffeeBreak();

contract Counter {
    uint256 public number;

    function setNumber(uint256 newNumber, bytes32 inLuck) public {
        number = newNumber;
        if (newNumber == 0xC0FFEE && inLuck == bytes32(hex"c0c0")) {
            revert CoffeeBreak();
        }
    }

    function increment() public {
        number++;
    }
}
```

Additionally, we need to update `Counter.t.sol`:

```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.13;

import {Test, console} from "forge-std/Test.sol";
import {Counter} from "../src/Counter.sol";
import {KontrolCheats} from "kontrol-cheatcodes/KontrolCheats.sol";

contract CounterTest is Test, KontrolCheats {
    Counter public counter;

    function setUp() public {
        counter = new Counter();
    }

    function testIncrement() public {
        counter.increment();
        assert(counter.number() == 1);
    }

    function testSetNumber(uint256 x, bytes32 inLuck) public {
        kevm.symbolicStorage(address(counter));
        counter.setNumber(x, bytes32(inLuck));
        assert(counter.number() == x);
    }
}
```

Once these changes have been made, you will need to rebuild the proof by running:&#x20;

```
kontrol build --regen --rekompile
```

Following that, you need to rerun `prove` with the following command:

```
kontrol prove --match-test CounterTest.testSetNumber --reinit
```

You can now look at the directory and you will see the newly generated proofs!

<figure><img src="../../.gitbook/assets/Untitled.png" alt=""><figcaption></figcaption></figure>

You will also see the new proof if you use the `kontrol list` command. However, the previous proof will no longer be listed. This does not mean that the previous proof no longer exists. You can still find it in the directory; it will simply not be displayed in the CLI.

<figure><img src="../../.gitbook/assets/Screenshot 2024-03-05 at 7.39.16 PM.png" alt=""><figcaption><p>Kontrol list</p></figcaption></figure>

## View Proofs in KCFG

Having multiple proofs will affect the command required to view the **KCFG**. If you have executed multiple proofs and attempt to run the following command:&#x20;

```
kontrol view-kcfg CounterTest.testSetNumber
```

You will likely receive a response similar to the one below:&#x20;

<figure><img src="../../.gitbook/assets/Screenshot 2024-03-05 at 7.40.19 PM.png" alt=""><figcaption><p>3 Matching Proofs</p></figcaption></figure>

This indicates that there are multiple proofs matching that name, and you will need to choose a version of the proof. To view a specific proof, you will need to run the following command:

```
kontrol view-kcfg 'CounterTest.testSetNumber(uint256,bytes32)' --version 0
```

The version number specifies which proof you intend to run. Additionally, you must ensure that the variables in the function match the values of the desired proof.&#x20;
