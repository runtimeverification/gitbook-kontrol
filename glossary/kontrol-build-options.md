# Kontrol Build Options

| Option                                                 | Description                                                                   |
| ------------------------------------------------------ | ----------------------------------------------------------------------------- |
| `-h`, `--help`                                         | Show this help message and exit                                               |
| `--verbose`, `-v`                                      | Verbose output                                                                |
| `--debug`                                              | Debug output                                                                  |
| `-I INCLUDES`                                          | Directories to lookup K definitions in                                        |
| `--main-module MAIN_MODULE`                            | Name of the main module                                                       |
| `--syntax-module SYNTAX_MODULE`                        | Name of the syntax module                                                     |
| `--spec-module SPEC_MODULE`                            | Name of the spec module                                                       |
| `--definition DEFINITION_DIR`                          | Path to definition to use                                                     |
| `--md-selector MD_SELECTOR`                            | Code selector expression to use when reading markdown                         |
| `--depth DEPTH`                                        | Maximum depth to execute to                                                   |
| `--require REQUIRES`                                   | Extra K requires to include in generated output                               |
| `--module-import IMPORTS`                              | Extra modules to import into generated main module                            |
| `--emit-json`                                          | Emit JSON definition after compilation                                        |
| `--no-emit-json`                                       | Do not JSON definition after compilation                                      |
| `-ccopt CCOPTS`                                        | Additional arguments to pass to llvm-kompile                                  |
| `--no-llvm-kompile`                                    | Do not run llvm-kompile process                                               |
| `--with-llvm-library`                                  | Make kompile generate a dynamic llvm library                                  |
| `--enable-llvm-debug`                                  | Make kompile generate debug symbols for llvm                                  |
| `--read-only-kompiled-directory`                       | Generated a kompiled directory that K will not attempt to write to afterwards |
| `-O0`                                                  | Optimization level 0                                                          |
| `-O1`                                                  | Optimization level 1                                                          |
| `-O2`                                                  | Optimization level 2                                                          |
| `-O3`                                                  | Optimization level 3                                                          |
| `--foundry-project-root FOUNDRY_ROOT`                  | Path to Foundry project root directory                                        |
| `--target {KompileTarget.HASKELL,KompileTarget.MAUDE}` | \[haskell\|maude]                                                             |
| `--regen`                                              | Regenerate foundry.k even if it already exists                                |
| `--rekompile`                                          | Rekompile foundry.k even if kompiled definition already exists                |
| `--no-forge-build`                                     | Do not call 'forge build' during kompilation                                  |
