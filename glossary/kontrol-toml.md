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

# ? definition_dir = None
# ? spec_module = None

[build]
# Directories to lookup K definitions in.
# include = []
# Name of the main module.
# main-module = None
# Name of the syntax module.
# syntax-module = None
# Code selector expression to use when reading markdown.
md-selector = 'k'
# Maximum depth to execute to.
# depth = None
# Path to kompile definition to.
# output-definition = # Cannot find the default value
# K backend to target with compilation.
# backend = # Cannot find the default value
# Mode for doing K rule type inference in.
# type-inference-mode = 
# Mode for doing K rule type inference in.
emit-json = true
# Additional arguments to pass to llvm-kompile.
ccopt = []
# Do not run llvm-kompile process.
no-llvm-kompile = false
# Make kompile generate a dynamic llvm library.
with-llvm-library  = false
# Make kompile generate debug symbols for llvm.
enable-llvm-debug = false
# Mode to kompile LLVM backend in.
# llvm-kompile-type = None
# Location to put kompiled LLVM backend at.
# llvm-kompile-output = None
# Generated a kompiled directory that K will not attempt to write to afterwards.
read-only-kompiled-directory = false
# Optimization level 0-3
O0 = false
O1 = false
O2 = false
O3 = false
# Enable search mode on LLVM backend krun.
enable-search = false
# Enable logging semantic rule coverage measurement.
coverage = false
# Generate standalone Bison parser for program sort.
gen-bison-parser = false
# Generate standalone GLR Bison parser for program sort.
gen-glr-bison-parser = false
# Disable List{Sort} parsing to make grammar LR(1) for Bison parser.
bison-lists = false
# Enable proof hint generation in LLVM backend kompilation.
llvm-proof-hint-instrumentation = false
# Enable additional proof hint debugging information in LLVM backend kompilation.
llvm-proof-hint-debugging = false
# Do not wrap the output on the CLI.
no-exc-wrap = false
# Ignore provided warnings
ignore-warnings = []
# Add constraints for enum function arguments and storage slots.
enum-constraints = false
# Target to compile to.
target = 'haskell'
# Path to Pyk config file.
# config-file =
# Config profile to be used.
config-profile = 'default' 
# Do not call 'forge build' during kompilation.
no-forge-build = false
# Do not silence K compiler warnings.
no-silence-warnings = false
# Do not append cbor or bytecode_hash metadata to bytecode.
no-metadata = false 
# Do not include assumptions on keccak properties.
no-keccak-lemmas = false

[prove.default]
foundry-project-root       = '.'
verbose                    = false
debug                      = false
# Maximum number of K steps before the state is saved in a new node in the CFG.
# Branching will cause this to happen earlier.
max-depth                  = 25000
# Reinitialize CFGs even if they already exist.
reinit                     = false
# Use Compositional Symbolic Execution
cse                        = false
# Number of processes to run in parallel.
workers                    = 4
# Show failure summary for all failing tests (default).
failure-information        = true
# Show models for failing nodes (default).
counterexample-information = true
# Minimize obtained KCFGs
minimize-proofs            = false
# Stop execution on other branches if a failing node is detected (default).
fail-fast                  = true
# Timeout in ms to use for SMT queries.
smt-timeout                = 1000
# Store a node for every EVM opcode step (expensive).
break-every-step           = false
# Store a node for every EVM jump opcode.
break-on-jumpi             = false
# Store a node for every EVM call made.
break-on-calls             = false
# Store a node for every EVM SSTORE/SLOAD made.
break-on-storage           = false
# Store a node for every EVM basic block (implies --break-on-calls).
break-on-basic-blocks      = false
# Break on all Foundry rules.
break-on-cheatcodes        = false
# Include the contract constructor in the test execution.
run-constructor            = false
# Optimize KEVM execution by removing stack overflow/underflow checks.Assumes
# running Solidity-compiled bytecode cannot result in a stack overflow/underflow.
no-stack-checks            = true

[prove]
  -I INCLUDES           Directories to lookup K definitions in.
  --main-module MAIN_MODULE
                        Name of the main module.
  --syntax-module SYNTAX_MODULE
                        Name of the syntax module.
  --md-selector MD_SELECTOR
                        Code selector expression to use when reading markdown.
# Maximum depth to execute to.
depth = # DEPTH
# Comma-separated list of equations to debug.
debug-equations = [] # DEBUG_EQUATIONS
# Use fast-check on k-cell to determine subsumption (experimental).
fast-check-subsumption = false
# For passing subproofs, construct lemmas directly from initial to target state.
direct-subproof-rules = false
# The number of proof iterations performed between two writes to disk and status bar updates. Note that setting to >1 may result in work being discarded if proof is interrupted.
maintenance-rate = 1 # MAINTENANCE_RATE
# Use the implication check of the Booster (experimental).
assume-defined = false
# Number of times to retry SMT queries with scaling timeouts.
smt-retry-limit = 10 # SMT_RETRY_LIMIT
# Z3 tactic to use when checking satisfiability. Example: (check-sat-using smt)
smt-tactic = # SMT_TACTIC
  --no-log-rewrites     Do not log traces of any simplification and rewrite rule application.
# log_succ_rewrites = true
# Log traces of all simplification and rewrite rule applications.
log-fail-rewrites = false
# Custom command to start RPC server.
kore-rpc-command = # KORE_RPC_COMMAND
# Use the booster RPC server instead of kore-rpc (default).
use-booster = true
  --no-use-booster      Do not use the booster RPC server instead of kore-rpc.
# Use existing RPC server on named port.
port = # PORT
# Use existing Maude RPC server on named port.
maude-port = # MAUDE_PORT
# Generate bug report with given name
bug-report = # BUG_REPORT
  --symbolic-immutables
                        Enable support for symbolic immutable variables in Solidity code.
# Number of times to expand the next pending node in the CFG.
# max-iterations = # MAX_ITERATIONS
# Automatically extract gas cell when infinite gas is enabled.
auto-abstract-gas = false
# Use sequential, single-threaded proof loop.
force-sequential = false
# Add constraints for enum function arguments and storage slots.
enum-constraints = false
# schedule to use for execution [DEFAULT|FRONTIER|HOMESTEAD|TANGERINE_WHISTLE|SPURIOUS_DRAGON|BYZANTIUM|CONSTANTINOPLE|PETERSBURG|ISTANBUL|BERLIN|LONDON|MERGE|SHANGHAI|CANCUN].
schedule = 'SHANGHAI' # {DEFAULT,FRONTIER,HOMESTEAD,TANGERINE_WHISTLE,SPURIOUS_DRAGON,BYZANTIUM,CONSTANTINOPLE,PETERSBURG,ISTANBUL,BERLIN,LONDON,MERGE,SHANGHAI,CANCUN}
# chain ID to use for execution.
chainid = 1 # CHAINID
# execution mode to use [{'|'.join(modes)}].
mode = 'NORMAL' # {NORMAL,VMTESTS}
  --no-gas              omit gas cost computations.
# Path to Pyk config file.
# config-file = # CONFIG_FILE
# Config profile to be used.
config-profile = 'default' # CONFIG_PROFILE
# Specify contract function(s) to test using a regular expression. This will match functions based on their full signature, e.g., 'ERC20Test.testTransfer(address,uint256)'. This option can be used multiple times to add more functions to test.
# match-test = []
# Instead of reinitializing the test setup together with the test proof, select the setup version to be reused during the proof.
setup-version = # SETUP_VERSION
# Maximum worker threads to use on a single proof to explore separate branches in parallel.
max-frontier-parallel = 1 # MAX_FRONTIER_PARALLEL
# Enables bounded model checking. Specifies the maximum depth to unroll all loops to.
bmc-depth = # BMC_DEPTH
# Enables gas computation in KEVM.
use-gas = false
# Config type
config-type = 'TEST_CONFIG' # {ConfigType.TEST_CONFIG,ConfigType.SUMMARY_CONFIG}
# Disables the proof status bar.
hide-status-bar = false
# Path to JSON file produced by vm.stopAndReturnStateDiff.
init-node-from-diff = # RECORDED_DIFF_STATE_PATH
# Path to JSON file produced by vm.dumpState.
init-node-from-dump = # RECORDED_DUMP_STATE_PATH
# Specify a summary to include as a lemma.
include-summary = [] # INCLUDE_SUMMARIES
# Flag used by Simbolik to initialise the state of a non test function as if it was a test function.
with-non-general-state = false
# Generate a JUnit XML report
xml-test-report = false
# Use hevm success predicate instead of foundry to determine if a test is passing
hevm = false 
# Trace opcode execution and store it in the configuration
evm-tracing = false
# If tracing is active, avoid storing storage information.
no-trace-storage = true
# If tracing is active, avoid storing wordstack information.
no-trace-wordstack = true
# If tracing is active, avoid storing memory information.
no-trace-memory = true
# Remove all outdated KCFGs.
remove-old-proofs = false
# Optimize performance for proof execution. Takes the number of parallel threads to be used.Will overwrite other settings of 'assume-defined', 'log-success-rewrites', 'max-frontier-parallel','maintenance-rate', 'smt-timeout', 'smt-retry-limit', 'max-depth', 'max-iterations', and 'no-stack-checks'.
optimize-performance = # OPTIMIZE_PERFORMANCE


[show.default]
foundry-project-root       = '.'
verbose                    = false
debug                      = false
use-hex-encoding           = false

[show]
# ??? 'version': None --- FoundryTestOptions
# Name of the main module.
# main-module = # MAIN_MODULE
# Name of the syntax module.
# syntax-module = # SYNTAX_MODULE
# Code selector expression to use when reading markdown.
md-selector = 'k' # MD_SELECTOR
# Maximum depth to execute to.
depth = # DEPTH
# List of nodes to display as well.
node = [] # NODES
# List of nodes to display delta for.
node-delta = [] # NODE_DELTAS
# Show failure summary for cfg.
failure-information = false
# Do not show failure summary for cfg.
no-failure-information = false
# Output edges as a K module. 
to-module = false
  --pending             Also display pending nodes.
  --failing             Also display failing nodes.
# Show models for failing nodes. Should be called with the '--failure-information' flag.
counterexample-information = true
# Minimize output.
minimize = true
# Do not minimize output.
no-minimize = false
# Sort collections before outputting term.
sort-collections = false
# Add constraints for enum function arguments and storage slots.
enum-constraints = false
# Path to Pyk config file.
config-file = # CONFIG_FILE
# Config profile to be used.
config-profile = 'default' # CONFIG_PROFILE
# Strip output that is likely to change without the contract logic changing
omit-unstable-output = false
# Generate a K module which can be run directly as KEVM claims for the given KCFG(best-effort).
to-kevm-claims = false
# Generate a K module which can be used to optimize KEVM execution (best-effort).
to-kevm-rules = false
# Path to write KEVM claim files at.
kevm-claim-dir = # KEVM_CLAIM_DIR
# When printing nodes, always show full bytecode in code and program cells, and do not hide jumpDests cell.
expand-config = false
# Run KCFG minimization routine before displaying it.
minimize-kcfg = false


[view-kcfg.default]
foundry-project-root       = '.'
use-hex-encoding           = false

[view-kcfg]
# Version of the test to use
version = # VERSION
# Verbose output.
verbose = false
# Debug output.
debug = false
# Add constraints for enum function arguments and storage slots.
enum-constraints = false
# Path to Pyk config file.
config-file = # CONFIG_FILE
# Config profile to be used.
config-profile = 'default' # CONFIG_PROFILE
```
