# Kup Cheatsheet

## K Framework Installer

All **K**-related tools are managed through the `kup` [tool](https://github.com/runtimeverification/kup).

The following commands are used to install the **K** framework and enable easy switching between different versions.

### **Install `kup`**

```bash
bash <(curl https://kframework.org/install)
```

### `kup` commands

<table data-full-width="false"><thead><tr><th>Command</th><th>Description</th><th>Example</th></tr></thead><tbody><tr><td><code>kup list</code></td><td>List all available packages and their status </td><td><code>kup list</code></td></tr><tr><td><code>kup install $package</code></td><td>Install <code>$package</code></td><td><code>kup install kevm</code></td></tr><tr><td><code>kup update $package</code></td><td>Update <code>$package</code> to the latest commit </td><td><code>kup update kevm</code></td></tr><tr><td><code>kup $option --help</code></td><td>Functionality description</td><td><code>kup list --help</code></td></tr></tbody></table>

**Installation time:** The initial installation of certain packages (e.g., [**kevm**](https://github.com/runtimeverification/evm-semantics)) may take longer as it needs to fetch all the libraries and compile sources. This process typically takes around **30mins** to **1 hour**.

### `kup` packages management

The following flags allow for the installation of different package versions and/or different dependencies versions.

<table data-full-width="true"><thead><tr><th width="154">Flag</th><th width="231">Usage</th><th width="227">Description</th><th width="191">Parameters</th><th>Example</th></tr></thead><tbody><tr><td><code>--version</code></td><td><code>kup update $package --version $version</code></td><td>Update <code>$package</code> with a particular <code>$version</code></td><td><code>$version</code>: Commit/branch/local checkout of <code>$package</code></td><td><code>kup update kevm --version ~/evm-semantics</code></td></tr><tr><td><code>--override</code></td><td><code>kup update $package --override $dependency $version</code></td><td>Update <code>$package</code> with a particular <code>$version</code> of <code>$dependency</code></td><td><code>$dependency</code>: a dependency of <code>$package</code><br><code>$version</code>: commit/branch/local checkout of <code>$dependency</code></td><td><code>kup update kevm --override k-framework/haskell-backend ~/haskell-backend</code></td></tr></tbody></table>

### **Chaining flags**

As an example, let'd assume we want to use `kevm` with the following modifications:

* Use a local checkout of `kevm` (for example, we wrote a lemma to help `kevm` reason better).
* Use the `haskell-backend` branch (for example, some execution improvements have not yet been upstreamed to `kevm`).
* Use a local checkout of `pyk` (for example, you're trying a new proof-strategy optimization).

The line below will allow us to run a version of `kevm` with the above modifications:

{% code overflow="wrap" %}
```bash
kup update kontrol --version ./$path_to_local_kevm --override k-framework/haskell-backend $haskell-branch pyk ./$path_to_local_pyk
```
{% endcode %}
