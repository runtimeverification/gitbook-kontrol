# Using structured symbolic storage in Kontrol tests

## Preliminaries

### How is storage organised?

EVM storage is organised at a per-contract level, with each contract having `2 ^Int 256` 32-byte storage slots available. Each slot can hold multiple pieces of data and understanding which storage slot corresponds to which part of contract data can get fairly complex, especially in the case of mappings and dynamically sized data. The detailed official guide to how storage slots are organised and computed can be found [here](https://docs.soliditylang.org/en/v0.8.24/internals/layout_in_storage.html).

### How are storage slot updates executed in EVM?

A general slot update in EVM is of the form
```k
(SHIFT:Int *Int VALUE:Int) |Int (MASK:Int &Int #asWord(SLOT:Bytes))
```
where:
- `SLOT` is a byte array representing the slot to be updated;
- `VALUE` is an integer representing the value to which a contract field in that slot should be updated;
- `MASK` is a bit-mask that is used to zero the part of the slot corresponding to the contract field to be updated; and
- `SHIFT` is a power of two that is used to shift the value so that it aligns with the part of the slot corresponding to the contract field to be updated.

For example, if we were updating a given 8-byte `uint64` field stored in a given slot starting from byte `8` (note that byte indices are computed from right to left), this would be achieved as follows, taking:
- `MASK ==Int 115792089237316195423570985008687907852929702298719625576012656144555070980095`, that is, `ffffffffffffffffffffffffffffffff0000000000000000ffffffffffffffff` in hex; and
- `SHIFT ==Int 2 ^Int 32`
```
32                      8        0
|xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx| : SLOT
|xxxxxxxxxxxxxxxx00000000xxxxxxxx| : MASK &Int #asWord(SLOT)

|000000000000000000000000yyyyyyyy| : VALUE
|0000000000000000yyyyyyyy00000000| : SHIFT *Int VALUE

|xxxxxxxxxxxxxxxxyyyyyyyyxxxxxxxx| : (SHIFT *Int VALUE) |Int (MASK &Int #asWord( SLOT:Bytes ))
```

### Where can we find storage layout information for real-world contracts?

The Solidity compiler is able to provide complete information on the storage layout of each of the compiled contracts. This information can be found in the `"storageLayout"` field of the associated `.json` file, which is created if the `'storageLayout'` option is passed to the `extra_output` parameter in compilation.

## Running example: `DualGovernance`

As our running example, we take the `DualGovernance` contract from our recent Lido engagement, taken from a relevant commit at the time of writing this guide, and focusing only on the part relevant for symbolic storage:

```solidity
// ---
// Aspects
// ---

/// @dev The functionality to manage the proposer -> executor pairs.
Proposers.Context internal _proposers;

/// @dev The functionality of the tiebreaker to handle "deadlocks"
/// of the Dual Governance.
Tiebreaker.Context internal _tiebreaker;

/// @dev The state machine implementation controlling the state
/// of the Dual Governance.
DualGovernanceStateMachine.Context internal _stateMachine;

// ---
// Standalone State Variables
// ---

/// @dev The address of the Reseal Committee which is allowed to
/// "reseal" sealables paused for a limited period of time when
/// the Dual Governance proposal adoption is blocked.
address internal _resealCommittee;
```
where the three used contexts are defined as follows:
```solidity
/// @notice Context: Proposers
struct Context {
    address[] proposers;
    mapping(address proposer => ExecutorData) executors;
    mapping(address executor => uint256 usagesCount) executorRefsCounts;
}

/// @dev Context: Tiebreaker
struct Context {
    /// @dev slot0 [0..159]
    address tiebreakerCommittee;
    /// @dev slot0 [160..191]
    Duration tiebreakerActivationTimeout;
    /// @dev slot1 [0..255]
    EnumerableSet.AddressSet sealableWithdrawalBlockers;
}

/// @notice Context: DualGovernanceStateMachine
struct Context {
    /// @dev slot 0: [0..7]
    State state;
    /// @dev slot 0: [8..47]
    Timestamp enteredAt;
    /// @dev slot 0: [48..87]
    Timestamp vetoSignallingActivatedAt;
    /// @dev slot 0: [88..247]
    IEscrow signallingEscrow;
    /// @dev slot 0: [248..255]
    uint8 rageQuitRound;
    /// @dev slot 1: [0..39]
    Timestamp vetoSignallingReactivationTime;
    /// @dev slot 1: [40..79]
    Timestamp normalOrVetoCooldownExitedAt;
    /// @dev slot 1: [80..239]
    IEscrow rageQuitEscrow;
    /// @dev slot 2: [0..159]
    IDualGovernanceConfigProvider configProvider;
}
```
where the other relevant types are defined as follows
```solidity
type Duration is uint32;

struct AddressSet {
    Set _inner;
}

struct Set {
    bytes32[] _values;
    mapping(bytes32 value => uint256) _positions;
}

enum State {
    Unset,
    Normal,
    VetoSignalling,
    VetoSignallingDeactivation,
    VetoCooldown,
    RageQuit
}

type Timestamp is uint40;
```
and where two of the contexts are even annotated with the expected storage slot structure. Given this information, and the [official guide for storage slot structuring](https://docs.soliditylang.org/en/v0.8.24/internals/layout_in_storage.html), we would expect the following:

1. `Proposers.Context internal _proposers` starts from slot `0` and occupies three slots (`0`, `1`, and `2`), one per each field of the context, as mappings require a dedicated entire slot.

2. `Tiebreaker.Context internal _tiebreaker` starts from slot `3` and, according to the annotations, occupies two slots. However, this annotation is incorrect because even though an `address` and a `Duration` can be packed into a single slot, `EnumerableSet.AddressSet` occupies two slots on its own.

3. `DualGovernanceStateMachine.Context internal _stateMachine` starts from slot `6` and, according to the annotations, occupies three slots. As we can see, this data has been carefully arranged to minimize slot occupancy, as slot `0` contains five pieces of data.

4. `address internal _resealCommittee` occupies a single slot, which would be slot `9`.

Let us now compare our expectations with the information obtained after compilation, by inspecting the `storageLayout` field of the created `DualGovernance.json` file. We first find this:
```json
"storageLayout": {
  "storage": [
    {
      "astId": 141,
      "contract": "contracts/DualGovernance.sol:DualGovernance",
      "label": "_proposers",
      "offset": 0,
      "slot": "0",
      "type": "t_struct(Context)11088_storage"
    },
    {
      "astId": 145,
      "contract": "contracts/DualGovernance.sol:DualGovernance",
      "label": "_tiebreaker",
      "offset": 0,
      "slot": "3",
      "type": "t_struct(Context)11677_storage"
    },
    {
      "astId": 149,
      "contract": "contracts/DualGovernance.sol:DualGovernance",
      "label": "_stateMachine",
      "offset": 0,
      "slot": "6",
      "type": "t_struct(Context)8128_storage"
    },
    {
      "astId": 152,
      "contract": "contracts/DualGovernance.sol:DualGovernance",
      "label": "_resealCommittee",
      "offset": 0,
      "slot": "9",
      "type": "t_address"
    }
  ],
  ...
```
which tells us that our assessment was correct in terms of top-level slot allocation, as can be seen by the four `slot` fields. To go deeper into the structure, we can look up the `type` of each slot. For example, searching for `t_struct(Context)8128_storage` yields:
```json
"t_struct(Context)8128_storage": {
  "encoding": "inplace",
  "label": "struct DualGovernanceStateMachine.Context",
  "numberOfBytes": "96",
  "members": [
    {
      "astId": 8096,
      "contract": "contracts/DualGovernance.sol:DualGovernance",
      "label": "state",
      "offset": 0,
      "slot": "0",
      "type": "t_enum(State)8082"
    },
    {
      "astId": 8100,
      "contract": "contracts/DualGovernance.sol:DualGovernance",
      "label": "enteredAt",
      "offset": 1,
      "slot": "0",
      "type": "t_userDefinedValueType(Timestamp)18110"
    },
    {
      "astId": 8104,
      "contract": "contracts/DualGovernance.sol:DualGovernance",
      "label": "vetoSignallingActivatedAt",
      "offset": 6,
      "slot": "0",
      "type": "t_userDefinedValueType(Timestamp)18110"
    },
    {
      "astId": 8108,
      "contract": "contracts/DualGovernance.sol:DualGovernance",
      "label": "signallingEscrow",
      "offset": 11,
      "slot": "0",
      "type": "t_contract(IEscrow)5824"
    },
    {
      "astId": 8111,
      "contract": "contracts/DualGovernance.sol:DualGovernance",
      "label": "rageQuitRound",
      "offset": 31,
      "slot": "0",
      "type": "t_uint8"
    },
    {
      "astId": 8115,
      "contract": "contracts/DualGovernance.sol:DualGovernance",
      "label": "vetoSignallingReactivationTime",
      "offset": 0,
      "slot": "1",
      "type": "t_userDefinedValueType(Timestamp)18110"
    },
    {
      "astId": 8119,
      "contract": "contracts/DualGovernance.sol:DualGovernance",
      "label": "normalOrVetoCooldownExitedAt",
      "offset": 5,
      "slot": "1",
      "type": "t_userDefinedValueType(Timestamp)18110"
    },
    {
      "astId": 8123,
      "contract": "contracts/DualGovernance.sol:DualGovernance",
      "label": "rageQuitEscrow",
      "offset": 10,
      "slot": "1",
      "type": "t_contract(IEscrow)5824"
    },
    {
      "astId": 8127,
      "contract": "contracts/DualGovernance.sol:DualGovernance",
      "label": "configProvider",
      "offset": 0,
      "slot": "2",
      "type": "t_contract(IDualGovernanceConfigProvider)5732"
    }
  ]
}
```
from where we can confirm that the detailed information about slot allocation present in the comments of the definition of `DualGovernanceStateMachine.Context` is correct.

## Symbolic storage management in Kontrol

Kontrol provides users with several tools that enable the creation of management of symbolic storage, and we will be using them here to demonstrate how to structure a part of the symbolic storage of the `DualGovernance` contract given above.

**Various symbolic unsigned integers**. First, we recall that Kontrol comes with cheatcodes that allow users to allocate symbolic unsigned integers of specified width (in bytes, between `1` and `32`):
```solidity
function freshUInt(uint8) external returns (uint256);
```

**Fully symbolic storage.** Next, we have a cheatcode that makes the storage of a given address *completely symbolic*:
```solidity
function symbolicStorage(address) external;
```
overwriting *any previous content* that might have already been in said storage.

**General slot management.** Kontrol comes with two core functions  following that allow users to load/store information from/to a specific part of a given slot. Both of these use the general slot update mechanism
```k
(SHIFT:Int *Int VALUE:Int) |Int (MASK:Int &Int #asWord(SLOT:Bytes))
```
and rely on its automation in Kontrol. First, the function for loading slot data has the following signature:
```solidity
function _loadData(
    address contractAddress, uint256 slot,
    uint256 offset, uint256 width
) internal view returns (uint256 slotData)
```
which returns an unsigned integer that represents the information stored in slot `slot` of contract with address `contractAddress`, starting from offset `offset` (in bytes) and capturing `width` bytes. Its implementation is as follows:
```solidity
function _loadData(
    address contractAddress,
    uint256 slot,
    uint256 offset,
    uint256 width
) internal view returns (uint256 slotData) {
    // `offset` and `width` must not overflow the slot
    assert(offset + width <= 32);

    // Read slot value
    slotData = uint256(vm.load(contractAddress, bytes32(slot)));
    // Shift value by `offset` bytes to the right
    slotData = slotData >> (8 * offset);
    // Create the bit-mask for `width` bytes
    uint256 mask = 2 ** (8 * width) - 1;
    // and apply it to isolate the desired data
    slotData = mask & slotData;
}
```

Next, the function for storing slot data has the following signature:
```solidity
function _storeData(
    address contractAddress, uint256 slot,
    uint256 offset, uint256 width,
    uint256 value) internal
```
which stores the unsigned integer `value` in slot `slot` of contract with address `contractAddress`, starting from offset `offset` (in bytes) and considering `width` bytes. Its implementation is as follows:
```solidity
function _storeData(
    address contractAddress,
    uint256 slot,
    uint256 offset,
    uint256 width,
    uint256 value
) internal {
    // `offset` and `width` must not overflow the slot
    assert(offset + width <= 32);
    // and `value` must fit into the designated part
    assert(width == 32 || value < 2 ** (8 * width));

    // Construct slot update mask
    uint256 maskLeft = ~((2 ** (8 * (offset + width))) - 1);
    uint256 maskRight = (2 ** (8 * offset)) - 1;
    uint256 mask = maskLeft | maskRight;
    // Shift the value appropriately
    uint256 value = (2 ** (8 * offset)) * value;

    // Get current slot value
    uint256 slotValue = uint256(vm.load(contractAddress, bytes32(slot)));
    // update it
    slotValue = value | (mask & slotValue);
    // and store the updated value
    vm.store(contractAddress, bytes32(slot), bytes32(slotValue));
}
```
In addition to these very low-level functions, the users have higher-level functions to operate with, such as:
```solidity
function _loadUInt256(
    address contractAddress,
    uint256 slot
) internal view returns (uint256) {
    return _loadData(contractAddress, slot, 0, 32);
}

function _storeUInt256(
    address contractAddress,
    uint256 slot,
    uint256 value
) internal {
    _storeData(contractAddress, slot, 0, 32, value);
}
```

**Symbolic storage: `DualGovernance`.** Below, we show how we use all of the above mechanisms to symbolically structure the part of the storage of the `DualGovernance` contract that is relevant to our tests:
```solidity
function dualGovernanceStorageSetup(
    DualGovernance _dualGovernance,
    IEscrow _signallingEscrow,
    IEscrow _rageQuitEscrow,
    IDualGovernanceConfigProvider _config
) external {
    kevm.symbolicStorage(address(_dualGovernance));

    // Slot 6:
    uint256 currentState = kevm.freshUInt(1);
    // Cannot be Unset as dual governance was initialised
    vm.assume(currentState != 0);
    vm.assume(currentState <= 5);

    uint256 enteredAt = kevm.freshUInt(5);
    vm.assume(enteredAt <= block.timestamp);
    vm.assume(enteredAt < timeUpperBound);

    uint256 vetoSignallingActivationTime = kevm.freshUInt(5);
    vm.assume(vetoSignallingActivationTime <= block.timestamp);
    vm.assume(vetoSignallingActivationTime < timeUpperBound);

    uint256 rageQuitRound = kevm.freshUInt(1);
    vm.assume(rageQuitRound < type(uint8).max);

    _storeData(address(_dualGovernance), 6,  0,  1, currentState);
    _storeData(address(_dualGovernance), 6,  1,  5, enteredAt);
    _storeData(address(_dualGovernance), 6,  6,  5, vetoSignallingActivationTime);
    _storeData(address(_dualGovernance), 6, 11, 20, uint256(uint160(address(_signallingEscrow))));
    _storeData(address(_dualGovernance), 6, 31,  1, rageQuitRound);

    // Slot 7
    uint256 vetoSignallingReactivationTime = kevm.freshUInt(5);
    vm.assume(vetoSignallingReactivationTime <= block.timestamp);
    vm.assume(vetoSignallingReactivationTime < timeUpperBound);

    uint256 normalOrVetoCooldownExitedAt = kevm.freshUInt(5);
    vm.assume(normalOrVetoCooldownExitedAt <= block.timestamp);
    vm.assume(normalOrVetoCooldownExitedAt < timeUpperBound);

    _storeData(address(_dualGovernance), 7,  0,  5, vetoSignallingReactivationTime);
    _storeData(address(_dualGovernance), 7,  5,  5, normalOrVetoCooldownExitedAt);
    _storeData(address(_dualGovernance), 7, 10, 20, uint256(uint160(address(_rageQuitEscrow))));

    // Slot 8
    _storeData(address(_dualGovernance), 8, 0, 20, uint256(uint160(address(_config))));
}
```

### How can we improve?

There still exists room for considerable improvement of the way Kontrol handles symbolic slot updates, and here we discuss several such directions.

**Automatically using slot-related information.** Firstly, note that the `_loadData` and `_storeData` functions are error-prone, as the slots, offsets, and widths have to be provided manually by the user. At the same time, all of the required information already exists in the `.json` file of the compiled contract. Unfortunately, this information is not available directly programmatically outside of the contract in question---within the contract, we can reflect on it in assembly using `.slot` and `.offset`. Therefore, we would have to devise either a cheatcode-like mechanism (perhaps using the custom-step mechanism) or create automatically an additional Solidity file with all of the auxiliary functions associated with each contract.

**Array slots and mapping slots**. Next, we could benefit from automated computation of array slots and mapping slots. This already exists in some of the developments, for example:
```solidity
function hashedLocation(address key, uint256 storageSlot) public pure returns (bytes32) {
    return keccak256(abi.encode(key, bytes32(storageSlot)));
}
```
but could benefit from being fully integrated with the associated functionalities.