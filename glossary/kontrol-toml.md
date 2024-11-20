# Configuration Options for Kontrol TOML file

The following is a kontrol.toml file with all configuration options set.

```toml
[build.default]
foundry-project-root       = '.'
regen                      = false
rekompile                  = false
verbose                    = false
debug                      = false
require                    = 'lemmas.k'
module-import              = 'TestBase:KONTROL-LEMMAS'
auxiliary-lemmas           = true
# definition_dir = 
# spec_module =

[build]
# backend = # K backend to target with compilation.
# type-inference-mode = # Mode for doing K rule type inference in.
emit-json = true # Mode for doing K rule type inference in.
ccopt = [] # Additional arguments to pass to llvm-kompile.
no-llvm-kompile = false # Do not run llvm-kompile process.
# Make kompile generate a dynamic llvm library.
with-llvm-library  = false
# Make kompile generate debug symbols for llvm.
enable-llvm-debug = false
read-only-kompiled-directory = false # Generated a kompiled directory that K will not attempt to write to afterwards.
O0 = false # Optimization level 0
O1 = false # Optimization level 1
O2 = false # Optimization level 2
O3 = false # Optimization level 3
enable-search = false # Enable search mode on LLVM backend krun.
coverage = false # Enable logging semantic rule coverage measurement.
gen-bison-parser = false # Generate standalone Bison parser for program sort.
gen-glr-bison-parser = false # Generate standalone GLR Bison parser for program sort.
bison-lists = false # Disable List{Sort} parsing to make grammar LR(1) for Bison parser.
llvm-proof-hint-instrumentation = false # Enable proof hint generation in LLVM backend kompilation.
llvm-proof-hint-debugging = false # Enable additional proof hint debugging information in LLVM backend kompilation.
no-exc-wrap = false # Do not wrap the output on the CLI.
ignore-warnings = [] # Ignore provided warnings
enum-constraints = false # Add constraints for enum function arguments and storage slots.
target = 'haskell' # Target to compile to.
# config-file = # Path to Pyk config file.
config-profile = 'default' # Config profile to be used.
no-forge-build = false # Do not call 'forge build' during kompilation.
no-silence-warnings = false # Do not silence K compiler warnings.
no-metadata = false # Do not append cbor or bytecode_hash metadata to bytecode.
no-keccak-lemmas = false # Do not include assumptions on keccak properties.
# include = [] # Directories to lookup K definitions in.
# main-module = # Name of the main module.
# syntax-module = # Name of the syntax module.
md-selector = 'k' # Code selector expression to use when reading markdown.
# depth = # Maximum depth to execute to.
# output-definition = # Path to kompile definition to.
# llvm-kompile-type = # Mode to kompile LLVM backend in.
# llvm-kompile-output = # Location to put kompiled LLVM backend at.

[prove.default]
foundry-project-root       = '.'
verbose                    = false
debug                      = false
max-depth                  = 25000 # Maximum number of K steps before the state is saved in a new node in the CFG.
reinit                     = false # Reinitialize CFGs even if they already exist.
cse                        = false # Use Compositional Symbolic Execution
workers                    = 4 # Number of processes to run in parallel.
failure-information        = true # Show failure summary for all failing tests (default).
counterexample-information = true # Show models for failing nodes (default).
minimize-proofs            = false # Minimize obtained KCFGs
fail-fast                  = true # Stop execution on other branches if a failing node is detected (default).
smt-timeout                = 1000 # Timeout in ms to use for SMT queries.
break-every-step           = false # Store a node for every rewriting step.
break-on-jumpi             = false # Store a node for every EVM jump opcode.
break-on-calls             = false # Store a node for every EVM call made.
break-on-storage           = false # Store a node for every EVM SSTORE/SLOAD made.
break-on-basic-blocks      = false # Store a node for every EVM basic block (implies --break-on-calls).
break-on-cheatcodes        = false # Break on all Foundry rules.
run-constructor            = false # Include the contract constructor in the test execution.
no-stack-checks            = true # Optimize KEVM execution by removing stack overflow/underflow checks.Assumes running Solidity-compiled bytecode cannot result in a stack overflow/underflow.
fast-check-subsumption = false # Use fast-check on k-cell to determine subsumption (experimental).
# debug-equations = [] # Comma-separated list of equations to debug.
direct-subproof-rules = false # For passing subproofs, construct lemmas directly from initial to target state.
maintenance-rate = 1 # The number of proof iterations performed between two writes to disk and status bar updates. Note that setting to >1 may result in work being discarded if proof is interrupted.
assume-defined = false # Use the implication check of the Booster (experimental).
smt-retry-limit = 10 # Number of times to retry SMT queries with scaling timeouts.
# smt-tactic = # Z3 tactic to use when checking satisfiability. Example: (check-sat-using smt)
log-rewrites = true # Log traces of all simplification and rewrite rule applications.
log-fail-rewrites = false # Log traces of all simplification and rewrite rule applications.
# kore-rpc-command = # Custom command to start RPC server.
use-booster = true # Use the booster RPC server instead of kore-rpc (default).
# port = # Use existing RPC server on named port.
# maude-port = # Use existing Maude RPC server on named port.
# bug-report = # Generate bug report with given name
# max-iterations = # Number of times to expand the next pending node in the CFG.
auto-abstract-gas = false # Automatically extract gas cell when infinite gas is enabled.
force-sequential = false # Use sequential, single-threaded proof loop.
enum-constraints = false # Add constraints for enum function arguments and storage slots.
schedule = 'SHANGHAI' # schedule to use for execution {DEFAULT,FRONTIER,HOMESTEAD,TANGERINE_WHISTLE,SPURIOUS_DRAGON,BYZANTIUM,CONSTANTINOPLE,PETERSBURG,ISTANBUL,BERLIN,LONDON,MERGE,SHANGHAI,CANCUN}
chainid = 1 # chain ID to use for execution.
mode = 'NORMAL' # execution mode to use [{'|'.join(modes)}].
# config-file = # Path to Pyk config file.
config-profile = 'default' # Config profile to be used.
# match-test = [] # Specify contract function(s) to test using a regular expression. This will match functions based on their full signature, e.g., 'ERC20Test.testTransfer(address,uint256)'. This option can be used multiple times to add more functions to test.
# setup-version = # Instead of reinitializing the test setup together with the test proof, select the setup version to be reused during the proof.
max-frontier-parallel = 1 # Maximum worker threads to use on a single proof to explore separate branches in parallel.
# bmc-depth = # Enables bounded model checking. Specifies the maximum depth to unroll all loops to.
use-gas = false # Enables gas computation in KEVM.
config-type = 'TEST_CONFIG' # {ConfigType.TEST_CONFIG,ConfigType.SUMMARY_CONFIG}
hide-status-bar = false # Disables the proof status bar.
# init-node-from-diff = # Path to JSON file produced by vm.stopAndReturnStateDiff.
# init-node-from-dump = # Path to JSON file produced by vm.dumpState.
# include-summary = [] # Specify a summary to include as a lemma.
with-non-general-state = false # Flag used by Simbolik to initialise the state of a non test function as if it was a test function.
xml-test-report = false # Generate a JUnit XML report
hevm = false # Use hevm success predicate instead of foundry to determine if a test is passing
evm-tracing = false # Trace opcode execution and store it in the configuration
no-trace-storage = true # If tracing is active, avoid storing storage information.
no-trace-wordstack = true # If tracing is active, avoid storing wordstack information.
no-trace-memory = true # If tracing is active, avoid storing memory information.
remove-old-proofs = false # Remove all outdated KCFGs.
# optimize-performance = # Optimize performance for proof execution. Takes the number of parallel threads to be used.Will overwrite other settings of 'assume-defined', 'log-success-rewrites', 'max-frontier-parallel','maintenance-rate', 'smt-timeout', 'smt-retry-limit', 'max-depth', 'max-iterations', and 'no-stack-checks'.


[show.default]
foundry-project-root       = '.'
verbose                    = false
debug                      = false
use-hex-encoding           = false

[show]
# node = [] # List of nodes to display as well.
# node-delta = [] # List of nodes to display delta for.
failure-information = false # Show failure summary for cfg.
no-failure-information = false # Do not show failure summary for cfg.
to-module = false # Output edges as a K module.
# pending = # Also display pending nodes.
# failing = # Also display failing nodes.
counterexample-information = true # Show models for failing nodes. Should be called with the '--failure-information' flag.
minimize = true # Minimize output.
no-minimize = false # Do not minimize output.
sort-collections = false # Sort collections before outputting term.
enum-constraints = false # Add constraints for enum function arguments and storage slots.
# config-file = # Path to Pyk config file.
# config-profile = 'default' # Config profile to be used.
omit-unstable-output = false # Strip output that is likely to change without the contract logic changing
to-kevm-claims = false # Generate a K module which can be run directly as KEVM claims for the given KCFG(best-effort).
to-kevm-rules = false # Generate a K module which can be used to optimize KEVM execution (best-effort).
# kevm-claim-dir = # Path to write KEVM claim files at.
expand-config = false # When printing nodes, always show full bytecode in code and program cells, and do not hide jumpDests cell.
minimize-kcfg = false # Run KCFG minimization routine before displaying it.


[view-kcfg.default]
foundry-project-root       = '.'
use-hex-encoding           = false

[view-kcfg]
# version = # Version of the test to use
verbose = false # Verbose output.
debug = false # Debug output.
enum-constraints = false # Add constraints for enum function arguments and storage slots.
# config-file = # Path to Pyk config file.
config-profile = 'default' # Config profile to be used.
```
