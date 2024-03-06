---
description: How to run property tests with Kontrol
---

# Kontrol Example

## Create a new Foundry project

```
forge init kontrolexample
```

This command creates a new Foundry project that serves as an example. The project's structure is explained in detail in the [Foundry book](https://book.getfoundry.sh/projects/project-layout). Currently, we can only perform fuzzing on parametric tests because the project is not configured to support symbolic execution. We will discuss this topic later in [property-verification-using-kontrol.md](property-verification-using-kontrol.md "mention"). With the project created, we will install **Kontrol** cheatcodes and then begin editing the code.

### Install Kontrol cheatcodes

To use **Kontrol** cheatcodes, we need to install a [new Solidity library](https://github.com/runtimeverification/kontrol-cheatcodes/) required for symbolic execution. First, **navigate** into the project directory. Then you can install it with Foundry by running the following command:

```
forge install runtimeverification/kontrol-cheatcodes
```

These cheatcodes enable us to generalize the storage of an Ethereum account by making it symbolic or by allowing any type of call, such as a [`delegatecall`](https://www.evm.codes/#f4).

{% hint style="info" %}
Adding custom cheatcodes will prevent us from running the tests with `forge test`, as `forge` will not recognize these cheatcodes.
{% endhint %}

With the project created and the **Kontrol** cheatcodes installed we can begin editing the code.
