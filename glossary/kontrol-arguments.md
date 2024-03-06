# Kontrol Arguments

| Positional Argument | Description                                                                 |
| ------------------- | --------------------------------------------------------------------------- |
| `version`           | Print out version of Kontrol command.                                       |
| `compile`           | Generate combined JSON with solc compilation results.                       |
| `solc-to-k`         | Output helper K definition for given JSON output from solc compiler.        |
| `build`             | Kompile K definition corresponding to given output directory.               |
| `load-state-diff`   | Generate a state diff summary from an account access dict                   |
| `prove`             | Run Foundry Proof.                                                          |
| `show`              | Print the CFG for a given proof.                                            |
| `to-dot`            | Dump the given CFG for the test as DOT for visualization.                   |
| `list`              | List information about CFGs on disk.                                        |
| `view-kcfg`         | Explore a given proof in the KCFG visualizer.                               |
| `remove-node`       | Remove a node and its successors.                                           |
| `refute-node`       | Refute a node and add its refutation as a subproof.                         |
| `unrefute-node`     | Disable refutation of a node and remove corresponding refutation subproof.  |
| `simplify-node`     | Simplify a given node, and potentially replace it.                          |
| `step-node`         | Step from a given node, adding it to the CFG.                               |
| `merge-nodes`       | Merge multiple nodes into one branch.                                       |
| `section-edge`      | Given an edge in the graph, cut it into sections to get intermediate nodes. |
| `get-model`         | Display a model for a given node.                                           |
