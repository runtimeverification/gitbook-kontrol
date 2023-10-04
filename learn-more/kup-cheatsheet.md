# Kup Cheatsheet

## K Framework Installer

All K-related tools are managed through the [`kup` package manager](https://github.com/runtimeverification/kup).

These commands are for installing the K framework and easily switching from different versions

### **Install `kup`**

```bash
bash <(curl https://kframework.org/install)
```

### `kup` commands

<table data-full-width="true"><thead><tr><th>Command</th><th>Description</th><th>Example</th></tr></thead><tbody><tr><td><code>kup list</code></td><td> List all available packages and their status </td><td><code>kup list</code></td></tr><tr><td><code>kup install $package</code></td><td>Install <code>$package</code></td><td><code>kup install kevm</code></td></tr><tr><td><code>kup update $package</code></td><td>Update <code>$package</code> to the latest commit </td><td><code>kup update kevm</code></td></tr><tr><td><code>kup $option --help</code></td><td>Functionality description</td><td><code>kup list --help</code></td></tr></tbody></table>

**Installation time:** The first installation of some packages (e.g., [kevm](https://github.com/runtimeverification/evm-semantics)) will take longer to fetch all the libraries and compile sources. (30m to 1h)

### `kup` packages management

The following flags allow the installation of different package versions and/or different dependencies versions.

<table data-full-width="true"><thead><tr><th>Flag</th><th width="186">Usage</th><th>Description</th><th>Parameters</th><th>Example</th></tr></thead><tbody><tr><td><code>--version</code></td><td><code>kup update $package --version $version</code></td><td>Update <code>$package</code> with a particular <code>$version</code></td><td><code>$version</code>: Commit/branch/local checkout of <code>$package</code></td><td><code>kup update kevm --version ~/evm-semantics</code></td></tr><tr><td><code>--override</code></td><td><code>kup update $package --override $dependency $version</code></td><td>Update <code>$package</code> with a particular <code>$version</code> of <code>$dependency</code></td><td><code>$dependency</code>: a dependency of <code>$package</code><br><code>$version</code>: commit/branch/local checkout of <code>$dependency</code></td><td><code>kup update kevm --override k-framework/haskell-backend ~/haskell-backend</code></td></tr></tbody></table>

**Chaining flags:** As an example, assume we want to use `kevm` with the following modifications

* Use a `kevm` local checkout (say, we wrote a lemma to help `kevm` reason better)
* Use a `haskell-backend` branch (say, some execution improvements have not yet been upstreamed to `kevm`)
* Use a `pyk` local checkout (say, you're trying a new proof-strategy optimization)

The following will allow us to have a running version of `kevm` with the above modifications:

{% code overflow="wrap" %}
```bash
kup update kontrol --version ./$path_to_local_kevm --override k-framework/haskell-backend $haskell-branch pyk ./$path_to_local_pyk
```
{% endcode %}
