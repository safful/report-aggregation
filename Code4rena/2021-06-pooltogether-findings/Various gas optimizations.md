## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Various gas optimizations](https://github.com/code-423n4/2021-06-pooltogether-findings/issues/77) 

# Handle

hrkrshnn


# Vulnerability details

# General Gas optimization

## Upgrade to at least 0.8.4 (even better is 0.8.5)

The following should lead to better gas savings:

  - The inliner should decrease runtime gas.
  - Inbuilt safemath instead of openzeppelin safemath should save some gas.
  - Various improvement in the expression simplifier in the compiler throughout (0.7.0 - 0.8.5)
    which should decrease both runtime and deploy time costs. (I'm assuming that the project
    currently uses 0.6.12, since the compiler version was not explicitly specified.)

Of course, these improvements comes when optimizer is enabled, preferably with a high
`--optimize-runs` value.

Note that the `inliner` in particular can be quite useful for the contract, since the contracts
sometimes generously chains small functions.

## Use custom errors instead of large revert strings

Saves both deploy time and runtime gas (runtime gas is only relevant when the revert condition is
met.)

Need at least solidity 0.8.4 for this feature.

### Use shorter revert strings

If you decide to not use custom errors, then try to use revert strings of size at most 32
characters.

For one, shorter strings would save deploy cost (one time saving of 200 gas per byte / character
decreased). Also strings more than 32 bytes requires an additional `mstore`, two additional `push`,
and an `add`. Roughly, 18 more gas during runtime (when revert condition is met).

Example string (33 bytes), from ControlledToken.sol

``` solidity
uint256 decreasedAllowance = allowance(_user, _operator).sub(_amount, "ControlledToken/exceeds-allowance");
```

# Specific Gas optimizations

## Use `immutable`

For state variables that are only assigned in constructors, change it to `immutable`.

This saves an `sload` each time the variable is accessed. Can save around 2100 gas (or 100 depending
on warm / cold.)

Examples:

### StakePrizePool.sol

``` diff
modified   contracts/StakePrizePool.sol
@@ -8,7 +8,7 @@ import "../PrizePool.sol";

 contract StakePrizePool is PrizePool {

-  IERC20Upgradeable private stakeToken;
+  IERC20Upgradeable immutable private stakeToken;

   event StakePrizePoolInitialized(address indexed stakeToken);
```

### ControlledToken.sol

``` diff
contract ControlledToken is ERC20PermitUpgradeable, ControlledTokenInterface {

   /// @notice Interface to the contract responsible for controlling mint/burn
-  TokenControllerInterface public override controller;
+  TokenControllerInterface public immutable override controller;
```

### yield-source/YearnV2YieldSource.sol

``` diff
@@ -24,7 +24,7 @@ contract YearnV2YieldSource is IYieldSource, ERC20Upgradeable, OwnableUpgradeabl
     /// @notice Yearn Vault which manages `token` to generate yield
     IYVaultV2 public vault;
     /// @dev Deposit Token contract address
-    IERC20Upgradeable internal token;
+    IERC20Upgradeable immutable internal token;
     /// @dev Max % of losses that the Yield Source will accept from the Vault in BPS
     uint256 public maxLosses = 0; // 100% would be 10_000
```

This change would likely require changing the initialization pattern. See the section below for
details.

Similarly, several such variables can be changed. Not listing everything here.

## Avoiding the `initialize` pattern

If elements can be initialized in the constructor, or via calls to internal functions in
constructor, instead of the public `initialize` function, it should be possible to save deployment
costs. On top of that, since the `initialize` function won't be part of the function dispatch in the
contract, one could save some gas at run time for some calls (saves approximately two `push`, an
`eq` and a `jumpi`.)

Another benefit for this is that several state variables can be converted to immutables. Again,
saves `sload` costs during runtime.

Also, it might also be possible to change `initialize` from `public` to `internal`.

## `_msgSender()` (Possible micro optimization)

Use `msg.sender` instead of `_msgSender()`. The latter might not be inlined by the compiler. (This
is for cases where `_msgSender()` function simply returns `msg.sender`.) Can save around 30 gas (2
`JUMP`, plus some `PUSH` and some stack operations.)

Also, the contracts seem to mix `_msgSender()` and `msg.sender`, for example in `PrizePool.sol`.
This could be avoided.

## Use `decreaseAllowance` in ControllerToken.sol

``` diff
@@ -58,8 +58,7 @@ contract ControlledToken is ERC20PermitUpgradeable, ControlledTokenInterface {
   /// @param _amount Amount of tokens to burn
   function controllerBurnFrom(address _operator, address _user, uint256 _amount) external virtual override onlyController {
     if (_operator != _user) {
-      uint256 decreasedAllowance = allowance(_user, _operator).sub(_amount, "ControlledToken/exceeds-allowance");
-      _approve(_user, _operator, decreasedAllowance);
+      decreaseAllowance(_user, _operator, _amount);
     }
     _burn(_user, _amount);
   }
```

Will be slightly more gas efficient than the first once.

# General comments

## Try to avoid `super` if possible

For example, in Ticket.sol:

``` solidity
  public
  virtual
  override
  initializer
{
  super.initialize(_name, _symbol, _decimals, _controller);
```

The above usage of `super` is unnecessary. Unless you are dealing with multiple inheritance, where
`super` is absolutely required, there is no need to use super, instead of statically specifying the
name of the parent contract. There is however no performance penalty in using `super` instead of a
static call to the parent.

## Several `balance` related function can be made `view`?

In PrizePool, the function `function balance() external returns (uint256)` can perhaps be made
`view`. This would also mean that a few other internal functions should be made `view`, such as
`_balance`.


