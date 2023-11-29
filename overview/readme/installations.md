---
description: Everything you need to install
---

# Installations

### Installing Foundry <a href="#h.nx9ig3q6eqt5" id="h.nx9ig3q6eqt5"></a>

To install Foundry execute the following command:

```
curl -L https://foundry.paradigm.xyz | bash
```

After installation open a new terminal session or reload your `PATH` and run `foundryup`.

For other installation methods, go to the [Foundry Documentation](https://book.getfoundry.sh/getting-started/installation).

### Installing Kontrol <a href="#h.c2tiycvv94xz" id="h.c2tiycvv94xz"></a>

The simplest way to install **Kontrol** is with the `kup`[ ](https://github.com/runtimeverification/kup)[package manager](https://github.com/runtimeverification/kup). To install `kup` execute the following command:

```
bash <(curl https://kframework.org/install)
```

After installing `kup`, install **Kontrol** using `kup` with the following command:

```
kup install kontrol
```

{% hint style="info" %}
The first installation of `kup` will take sometime. Check out the [kup-cheatsheet.md](../../cheatsheets/kup-cheatsheet.md "mention") for some additional information!
{% endhint %}

For detailed instructions on building **Kontrol** from source, go to the [**Kontrol** repository](https://github.com/runtimeverification/kontrol).
