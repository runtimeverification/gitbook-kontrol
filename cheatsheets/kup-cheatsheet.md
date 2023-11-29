# Kup Cheatsheet

## K Framework Installer

All **K**-related tools are managed through the `kup` [package manager](https://github.com/runtimeverification/kup). Below you will find the command to install the **K** framework, commands available with `kup` and flags that allow easy switching between different versions.

### **Install `kup`**

```bash
bash <(curl https://kframework.org/install)
```

### `kup` commands

<table data-full-width="false"><thead><tr><th>Command</th><th>Description</th><th>Example</th></tr></thead><tbody><tr><td><code>kup list</code></td><td>List all available packages and their status</td><td><code>kup list</code></td></tr><tr><td><code>kup install $package</code></td><td>Install or update a <code>$package</code></td><td><code>kup install kontrol</code></td></tr><tr><td><code>kup $option --help</code></td><td>Output the description of <code>$option</code></td><td><code>kup list --help</code></td></tr><tr><td><code>kup uninstall $package</code></td><td>Uninstall <code>$package</code></td><td><code>kup uninstall kontrol</code></td></tr></tbody></table>

**Installation time:** The initial installation of certain packages (e.g., [**kontrol**](https://github.com/runtimeverification/kontrol)) may take longer as it needs to fetch all the libraries and compile sources. This process typically takes around **30 mins** to **1 hour**.

### `kup` packages management

The following flags allow for the installation of different package versions and/or different dependencies versions.

<table data-full-width="true"><thead><tr><th width="154">Flag</th><th width="231">Usage</th><th width="227">Description</th><th width="191">Parameters</th><th>Example</th></tr></thead><tbody><tr><td><code>--version</code></td><td><code>kup install $package --version $version</code></td><td>Install/update <code>$package</code> with a particular <code>$version</code></td><td><code>$version</code>: Commit/branch/local checkout/release tag of <code>$package</code></td><td><code>kup install kontrol --version ~/kontrol</code></td></tr><tr><td><code>--override</code></td><td><code>kup install $package --override $dependency $version</code></td><td>Install/update <code>$package</code> with a particular <code>$version</code> of <code>$dependency</code></td><td><code>$dependency</code>: a dependency of <code>$package</code><br><code>$version</code>: commit/branch/local checkout/release tag of <code>$dependency</code></td><td><code>kup install kontrol --override kevm/k-framework/haskell-backend ~/haskell-backend</code></td></tr></tbody></table>

### **Chaining flags**

As an example, let's assume we want to use `kontrol` with the following modifications:

* Use a local checkout of `kontrol` (for example, after adding a new feature to `kontrol`).
* Use a `haskell-backend` branch (for example, some execution improvements have not yet been upstreamed to `kontrol`).
* Use the Github release `v0.1.461` of `pyk`, which includes a useful new feature.

The line below will allow us to run a version of `kontrol` with the above modifications:

{% code overflow="wrap" %}
```bash
kup install kontrol --version ./$path_to_local_kevm --override k-framework/haskell-backend $haskell-branch pyk ./$path_to_local_pyk
```
{% endcode %}

As you can see, release tags, local checkouts and branches can be used to build a fine tuned version of any of our tools, `kontrol` being the example here. To know the exact naming of the dependencies that a package has one can use `kup list $package --inputs`.

## Troubleshooting

### Making changes to nix.conf (manually or through `kup`) has no effect.

[Nix](https://nixos.org/) configuration files can be stored in many different locations. A full treatment can be found in the [Nix manual](https://nixos.org/manual/nix/stable/command-ref/conf-file). Check if you have additional Nix config files, for example in `$HOME/.config/nix`. If `$HOME/.config/nix` specifies a `trusted-users` option, this will override whichever `trusted-users` were set in `/etc/nix/nix.conf`. Changing the config variable from `trusted-users` to `extra-trusted-users` means it will only be appended, not overwritten. You can also explicitly set which files Nix should use as config files using the `NIX_USER_CONF_FILES` environment variable.

```
export NIX_USER_CONF_FILES = "/etc/nix/nix.conf"
kup doctor
```
