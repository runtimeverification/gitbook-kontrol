---
description: Everything you need to install
---

# Installations

### Installing Foundry <a href="#h.nx9ig3q6eqt5" id="h.nx9ig3q6eqt5"></a>

To install Foundry execute the following command:

```
curl -L https://foundry.paradigm.xyz | bash
```

After installation open a new terminal session or reload your PATH and run foundryup.

For other installation methods, go to the [Foundry Documentation](https://book.getfoundry.sh/getting-started/installation).

### Installing KEVM <a href="#h.c2tiycvv94xz" id="h.c2tiycvv94xz"></a>

The simplest way to install **KEVM** is with the `kup`[ tool](https://github.com/runtimeverification/kup). To install `kup` execute the following command:

```
bash <(curl https://kframework.org/install)
```

After installing `kup`, install **KEVM** using `kup` with the following command:

Note: The first installation of `kup` will take sometime.

```
kup install kevm
```

For detailed instructions on building **KEVM** from source, go to the [KEVM repository](https://github.com/runtimeverification/evm-semantics).
