## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Gas Optimizations](https://github.com/code-423n4/2022-04-axelar-findings/issues/14) 

# Gas Report

**Table of Contents:**

- [Gas Report](#gas-report)
  - [Foreword](#foreword)
  - [Findings](#findings)
    - [Variables](#variables)
      - [Constants should be literal and their derivation should be in comments](#constants-should-be-literal-and-their-derivation-should-be-in-comments)
    - [For-Loops](#for-loops)
      - [An array's length should be cached to save gas in for-loops](#an-arrays-length-should-be-cached-to-save-gas-in-for-loops)
      - [`++i` costs less gas compared to `i++` or `i += 1`](#i-costs-less-gas-compared-to-i-or-i--1)
      - [Increments can be unchecked](#increments-can-be-unchecked)
    - [Arithmetics](#arithmetics)
      - [Unchecking arithmetics operations that can't underflow/overflow](#unchecking-arithmetics-operations-that-cant-underflowoverflow)
    - [Errors](#errors)
      - [Use Custom Errors instead of Revert Strings to save Gas](#use-custom-errors-instead-of-revert-strings-to-save-gas)

## Foreword

- **`@audit` tags**

> The code is annotated at multiple places with `//@audit` comments to pinpoint the issues. Please, pay attention to them for more details.

## Findings

### Variables

#### Constants should be literal and their derivation should be in comments

Due to how `constant` variables are implemented (replacements at compile-time), an expression assigned to a `constant` variable is recomputed each time that the variable is used, which wastes some gas.

While I'm certain the sponsor is aware of this fact (thanks to the comments `AUDIT: constants should be literal and their derivation should be in comments`), I figured I'd still centralize the impacted lines of codes here:

```solidity
AdminMultisigBase.sol:16:    bytes32 internal constant KEY_ADMIN_EPOCH = keccak256('admin-epoch');
AdminMultisigBase.sol:18:    bytes32 internal constant PREFIX_ADMIN = keccak256('admin');
AdminMultisigBase.sol:19:    bytes32 internal constant PREFIX_ADMIN_COUNT = keccak256('admin-count');
AdminMultisigBase.sol:20:    bytes32 internal constant PREFIX_ADMIN_THRESHOLD = keccak256('admin-threshold');
AdminMultisigBase.sol:21:    bytes32 internal constant PREFIX_ADMIN_VOTE_COUNTS = keccak256('admin-vote-counts');
AdminMultisigBase.sol:22:    bytes32 internal constant PREFIX_ADMIN_VOTED = keccak256('admin-voted');
AdminMultisigBase.sol:23:    bytes32 internal constant PREFIX_IS_ADMIN = keccak256('is-admin');
AxelarGateway.sol:45:    bytes32 internal constant KEY_ALL_TOKENS_FROZEN = keccak256('all-tokens-frozen');
AxelarGateway.sol:47:    bytes32 internal constant PREFIX_COMMAND_EXECUTED = keccak256('command-executed');
AxelarGateway.sol:48:    bytes32 internal constant PREFIX_TOKEN_ADDRESS = keccak256('token-address');
AxelarGateway.sol:49:    bytes32 internal constant PREFIX_TOKEN_TYPE = keccak256('token-type');
AxelarGateway.sol:50:    bytes32 internal constant PREFIX_TOKEN_FROZEN = keccak256('token-frozen');
AxelarGateway.sol:51:    bytes32 internal constant PREFIX_CONTRACT_CALL_APPROVED = keccak256('contract-call-approved');
AxelarGateway.sol:52:    bytes32 internal constant PREFIX_CONTRACT_CALL_APPROVED_WITH_MINT = keccak256('contract-call-approved-with-mint');
AxelarGateway.sol:54:    bytes32 internal constant SELECTOR_BURN_TOKEN = keccak256('burnToken');
AxelarGateway.sol:55:    bytes32 internal constant SELECTOR_DEPLOY_TOKEN = keccak256('deployToken');
AxelarGateway.sol:56:    bytes32 internal constant SELECTOR_MINT_TOKEN = keccak256('mintToken');
AxelarGateway.sol:57:    bytes32 internal constant SELECTOR_APPROVE_CONTRACT_CALL = keccak256('approveContractCall');
AxelarGateway.sol:58:    bytes32 internal constant SELECTOR_APPROVE_CONTRACT_CALL_WITH_MINT = keccak256('approveContractCallWithMint');
AxelarGateway.sol:59:    bytes32 internal constant SELECTOR_TRANSFER_OPERATORSHIP = keccak256('transferOperatorship');
AxelarGateway.sol:60:    bytes32 internal constant SELECTOR_TRANSFER_OWNERSHIP = keccak256('transferOwnership');
AxelarGatewayMultisig.sol:25:    bytes32 internal constant KEY_OWNER_EPOCH = keccak256('owner-epoch');
AxelarGatewayMultisig.sol:27:    bytes32 internal constant PREFIX_OWNER = keccak256('owner');
AxelarGatewayMultisig.sol:28:    bytes32 internal constant PREFIX_OWNER_COUNT = keccak256('owner-count');
AxelarGatewayMultisig.sol:29:    bytes32 internal constant PREFIX_OWNER_THRESHOLD = keccak256('owner-threshold');
AxelarGatewayMultisig.sol:30:    bytes32 internal constant PREFIX_IS_OWNER = keccak256('is-owner');
AxelarGatewayMultisig.sol:32:    bytes32 internal constant KEY_OPERATOR_EPOCH = keccak256('operator-epoch');
AxelarGatewayMultisig.sol:34:    bytes32 internal constant PREFIX_OPERATOR = keccak256('operator');
AxelarGatewayMultisig.sol:35:    bytes32 internal constant PREFIX_OPERATOR_COUNT = keccak256('operator-count');
AxelarGatewayMultisig.sol:36:    bytes32 internal constant PREFIX_OPERATOR_THRESHOLD = keccak256('operator-threshold');
AxelarGatewayMultisig.sol:37:    bytes32 internal constant PREFIX_IS_OPERATOR = keccak256('is-operator');
```

As already proposed by a previous auditor, I suggest hardcoding the computed values from these expressions in the constants variables and add a comment above them to say how the value was calculated.

### For-Loops

#### An array's length should be cached to save gas in for-loops

Reading array length at each iteration of the loop takes 6 gas (3 for mload and 3 to place memory_offset) in the stack.  
  
Caching the array length in the stack saves around 3 gas per iteration.  

Here, I suggest storing the array's length in a variable before the for-loop, and use it instead:

```solidity
AxelarGatewayMultisig.sol:42:        for (uint256 i; i < accounts.length - 1; ++i) {
AxelarGatewayMultisig.sol:118:        for (uint256 i; i < accounts.length; i++) {
AxelarGatewayMultisig.sol:271:        for (uint256 i; i < accounts.length; i++) {

```

This is already done at most places in the solution.

#### `++i` costs less gas compared to `i++` or `i += 1`

`++i` costs less gas compared to `i++` or `i += 1` for unsigned integer, as pre-increment is cheaper (about 5 gas per iteration). This statement is true even with the optimizer enabled.

`i++` increments `i` and returns the initial value of `i`. Which means:  
  
```solidity
uint i = 1;  
i++; // == 1 but i == 2  
```
  
But `++i` returns the actual incremented value:  
  
```solidity
uint i = 1;  
++i; // == 2 and i == 2 too, so no need for a temporary variable  
```
  
In the first case, the compiler has to create a temporary variable (when used) for returning `1` instead of `2`  
  
Instances include:  

```solidity
AdminMultisigBase.sol:51:        for (uint256 i; i < adminCount; i++) {
AdminMultisigBase.sol:158:        for (uint256 i; i < adminLength; i++) {
AxelarGateway.sol:225:        for (uint256 i; i < adminCount; i++) {
AxelarGatewayMultisig.sol:118:        for (uint256 i; i < accounts.length; i++) {
AxelarGatewayMultisig.sol:140:        for (uint256 i; i < ownerCount; i++) {
AxelarGatewayMultisig.sol:181:        for (uint256 i; i < accountLength; i++) {
AxelarGatewayMultisig.sol:271:        for (uint256 i; i < accounts.length; i++) {
AxelarGatewayMultisig.sol:293:        for (uint256 i; i < operatorCount; i++) {
AxelarGatewayMultisig.sol:332:        for (uint256 i; i < accountLength; i++) {
AxelarGatewayMultisig.sol:495:        for (uint256 i; i < signatureCount; i++) {
AxelarGatewayMultisig.sol:526:        for (uint256 i; i < commandsLength; i++) {
```

I suggest using `++i` instead of `i++` to increment the value of an uint variable, just like it's done here:

```solidity
AxelarGatewayMultisig.sol:42:        for (uint256 i; i < accounts.length - 1; ++i) {
```

#### Increments can be unchecked

In Solidity 0.8+, there's a default overflow check on unsigned integers. It's possible to uncheck this in for-loops and save some gas at each iteration, but at the cost of some code readability, as this uncheck cannot be made inline.  
  
[ethereum/solidity#10695](https://github.com/ethereum/solidity/issues/10695)

Instances include:  

```solidity
AdminMultisigBase.sol:51:        for (uint256 i; i < adminCount; i++) {
AdminMultisigBase.sol:158:        for (uint256 i; i < adminLength; i++) {
AxelarGateway.sol:225:        for (uint256 i; i < adminCount; i++) {
AxelarGatewayMultisig.sol:42:        for (uint256 i; i < accounts.length - 1; ++i) {
AxelarGatewayMultisig.sol:118:        for (uint256 i; i < accounts.length; i++) {
AxelarGatewayMultisig.sol:140:        for (uint256 i; i < ownerCount; i++) {
AxelarGatewayMultisig.sol:181:        for (uint256 i; i < accountLength; i++) {
AxelarGatewayMultisig.sol:271:        for (uint256 i; i < accounts.length; i++) {
AxelarGatewayMultisig.sol:293:        for (uint256 i; i < operatorCount; i++) {
AxelarGatewayMultisig.sol:332:        for (uint256 i; i < accountLength; i++) {
AxelarGatewayMultisig.sol:495:        for (uint256 i; i < signatureCount; i++) {
AxelarGatewayMultisig.sol:526:        for (uint256 i; i < commandsLength; i++) {
```

The code would go from:  
  
```solidity
for (uint256 i; i < numIterations; i++) {  
 // ...  
}  
```

to:  

```solidity
for (uint256 i; i < numIterations;) {  
 // ...  
 unchecked { ++i; }  
}  
```

The risk of overflow is inexistant for a `uint256` here.

### Arithmetics  

#### Unchecking arithmetics operations that can't underflow/overflow

Solidity version 0.8+ comes with implicit overflow and underflow checks on unsigned integers. When an overflow or an underflow isn't possible (as an example, when a comparison is made before the arithmetic operation), some gas can be saved by using an `unchecked` block: <https://docs.soliditylang.org/en/v0.8.10/control-structures.html#checked-or-unchecked-arithmetic>

I suggest wrapping with an `unchecked` block here (see `@audit` tags for more details):

```solidity
src/AxelarGatewayMultisig.sol:
  103:         uint256 lowerBoundOwnerEpoch = epoch > recentEpochs ? epoch - recentEpochs : uint256(0); //@audit gas: should be unchecked due to condition in ternary operation
  257:         uint256 lowerBoundOperatorEpoch = epoch > recentEpochs ? epoch - recentEpochs : uint256(0); //@audit gas: should be unchecked due to condition in ternary operation
```

### Errors

#### Use Custom Errors instead of Revert Strings to save Gas

Custom errors from Solidity 0.8.4 are cheaper than revert strings (cheaper deployment cost and runtime cost when the revert condition is met)

Source: <https://blog.soliditylang.org/2021/04/21/custom-errors/>:
> Starting from [Solidity v0.8.4](https://github.com/ethereum/solidity/releases/tag/v0.8.4), there is a convenient and gas-efficient way to explain to users why an operation failed through the use of custom errors. Until now, you could already use strings to give more information about failures (e.g., `revert("Insufficient funds.");`), but they are rather expensive, especially when it comes to deploy cost, and it is difficult to use dynamic information in them.

Custom errors are defined using the `error` statement, which can be used inside and outside of contracts (including interfaces and libraries).

Instances include:

```solidity
BurnableMintableCappedERC20.sol:58:        require(!EternalStorage(owner).getBool(KEY_ALL_TOKENS_FROZEN), 'IS_FROZEN');
BurnableMintableCappedERC20.sol:59:        require(!EternalStorage(owner).getBool(keccak256(abi.encodePacked(PREFIX_TOKEN_FROZEN, symbol))), 'IS_FROZEN');
DepositHandler.sol:12:        require(_lockedStatus == IS_NOT_LOCKED);
ERC20.sol:164:        require(sender != address(0), 'ZERO_ADDR');
ERC20.sol:165:        require(recipient != address(0), 'ZERO_ADDR');
ERC20.sol:184:        require(account != address(0), 'ZERO_ADDR');
ERC20.sol:205:        require(account != address(0), 'ZERO_ADDR');
ERC20.sol:232:        require(owner != address(0), 'ZERO_ADDR');
ERC20.sol:233:        require(spender != address(0), 'ZERO_ADDR');
ERC20Permit.sol:43:        require(block.timestamp < deadline, 'EXPIRED');
ERC20Permit.sol:44:        require(uint256(s) <= 0x7FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF5D576E7357A4501DDFE92F46681B20A0, 'INV_S');
ERC20Permit.sol:45:        require(v == 27 || v == 28, 'INV_V');
ERC20Permit.sol:56:        require(recoveredAddress == issuer, 'INV_SIG');
MintableCappedERC20.sol:25:        require(capacity == 0 || totalSupply + amount <= capacity, 'CAP_EXCEEDED');
Ownable.sol:16:        require(owner == msg.sender, 'NOT_OWNER');
Ownable.sol:21:        require(newOwner != address(0), 'ZERO_ADDR');
```

I suggest replacing revert strings with custom errors.
