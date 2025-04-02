---
description: Everything you need to install
---

# Installations

## System Requirements

Before installing Kontrol, ensure your system meets the following requirements:

- **RAM**: 16GB of RAM is recommended for running Kontrol effectively
- **SWAP Space**: 16GB of SWAP space is recommended for installation and operation
  - On Linux systems, you can check your current SWAP space with `free -h`
  - To increase SWAP space, you can create a swap file:
    ```bash
    sudo fallocate -l 16G /swapfile
    sudo chmod 600 /swapfile
    sudo mkswap /swapfile
    sudo swapon /swapfile
    ```
    To make the SWAP file permanent, add this line to `/etc/fstab`:
    ```
    /swapfile none swap sw 0 0
    ```

{% hint style="warning" %}
Users running WSL (Windows Subsystem for Linux) should note that WSL allocates only 2GB of RAM by default. This is insufficient for Kontrol. You'll need to increase the memory allocation in your WSL configuration file (`.wslconfig`) to at least 16GB.
{% endhint %}

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

#### CI Installation

For GitHub Actions workflows, you can use the official [install-kontrol](https://github.com/runtimeverification/install-kontrol) action. Here's an example workflow:

```yaml
name: Kontrol CI
on: [push, pull_request]

jobs:
  verify:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '18'
      - uses: foundry-rs/foundry-toolchain@v1
      - uses: runtimeverification/install-kontrol@v1
      - name: Run Kontrol
        run: kontrol prove
```

This action handles the installation of all required dependencies and sets up Kontrol in your CI environment. Note that this workflow:
- Sets up Node.js (required for Kontrol)
- Installs Foundry (required dependency)
- Installs Kontrol
- Runs the verification

For detailed instructions on building **Kontrol** from source, go to the [**Kontrol** repository](https://github.com/runtimeverification/kontrol).
