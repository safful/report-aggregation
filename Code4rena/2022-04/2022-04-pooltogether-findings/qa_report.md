## Tags

- bug
- QA (Quality Assurance)
- sponsor confirmed

# [QA Report](https://github.com/code-423n4/2022-04-pooltogether-findings/issues/85) 

## Low Risk Issues

### 1. `safeApprove()` is deprecated
[Deprecated](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/bfff03c0d2a59bcd8e2ead1da9aed9edf0080d05/contracts/token/ERC20/utils/SafeERC20.sol#L38-L45) in favor of `safeIncreaseAllowance()` and `safeDecreaseAllowance()`. If only setting the initial allowance to the value that means infinite, `safeIncreaseAllowance()` can be used instead

```solidity
File: contracts/AaveV3YieldSource.sol   #1

183       IERC20(_tokenAddress()).safeApprove(address(_pool()), type(uint256).max);
```
https://github.com/pooltogether/aave-v3-yield-source/blob/e63d1b0e396a5bce89f093630c282ca1c6627e44/contracts/AaveV3YieldSource.sol#L183

### 2. Comments should be enforced with `require()`s
The comment below should be enforced with `require(decimals_ == _aToken.decimals())`. If this seems excessive, then why require `decimals_` be passed in at all? Why isn't `_aToken.decimals()` stored instead?

```solidity
File: contracts/AaveV3YieldSource.sol   #1

156     * @param decimals_ Number of decimals the shares (inhereted ERC20) will have. Same as underlying asset to ensure sane exchange rates for shares.
```
https://github.com/pooltogether/aave-v3-yield-source/blob/e63d1b0e396a5bce89f093630c282ca1c6627e44/contracts/AaveV3YieldSource.sol#L156

### 3. Formula does not match what the code is doing

```solidity
File: contracts/AaveV3YieldSource.sol   #1

360      // shares = (tokens * totalShares) / yieldSourceATokenTotalSupply
361      return _supply == 0 ? _tokens : _tokens.mul(_supply).div(aToken.balanceOf(address(this)));
```
should be `// shares = (tokens * totalSupply) / yieldSourceBalanceOfAToken`
https://github.com/pooltogether/aave-v3-yield-source/blob/e63d1b0e396a5bce89f093630c282ca1c6627e44/contracts/AaveV3YieldSource.sol#L360-L361

```solidity
File: contracts/AaveV3YieldSource.sol   #2

372      // tokens = (shares * yieldSourceATokenTotalSupply) / totalShares
373      return _supply == 0 ? _shares : _shares.mul(aToken.balanceOf(address(this))).div(_supply);
```
should be `// tokens = (shares * yieldSourceBalanceOfAToken) / totalSupply`
https://github.com/pooltogether/aave-v3-yield-source/blob/e63d1b0e396a5bce89f093630c282ca1c6627e44/contracts/AaveV3YieldSource.sol#L372-L373

### 4. Revert if amount is zero
There is already a check in one of the functions that the final token amount is not zero, but it would be better to check the input amount first in _all_ functions that take in an amount

```solidity
File: contracts/AaveV3YieldSource.sol   #1

231    function supplyTokenTo(uint256 _depositAmount, address _to) external override nonReentrant {
```
https://github.com/pooltogether/aave-v3-yield-source/blob/e63d1b0e396a5bce89f093630c282ca1c6627e44/contracts/AaveV3YieldSource.sol#L231

```solidity
File: contracts/AaveV3YieldSource.sol   #2

251    function redeemToken(uint256 _redeemAmount) external override nonReentrant returns (uint256) {
```
https://github.com/pooltogether/aave-v3-yield-source/blob/e63d1b0e396a5bce89f093630c282ca1c6627e44/contracts/AaveV3YieldSource.sol#L251

```solidity
File: contracts/AaveV3YieldSource.sol   #3

332    function transferERC20(
333      IERC20 _token,
334      address _to,
335      uint256 _amount
336    ) external onlyManagerOrOwner {
```
https://github.com/pooltogether/aave-v3-yield-source/blob/e63d1b0e396a5bce89f093630c282ca1c6627e44/contracts/AaveV3YieldSource.sol#L332-L336


## Non-critical Issues

### 1. Consider making whether to `safeApprove()` be based on a constructor argument
Approvals are only needed if doing flash loans or liquidations. If these are not used by the strategy, there is no need for the approval, which will lower the attack surface.

```solidity
File: contracts/AaveV3YieldSource.sol   #1

183      IERC20(_tokenAddress()).safeApprove(address(_pool()), type(uint256).max);
```
https://github.com/pooltogether/aave-v3-yield-source/blob/e63d1b0e396a5bce89f093630c282ca1c6627e44/contracts/AaveV3YieldSource.sol#L183

### 2. Function state mutability can be restricted to view
The compiler warns about this issue during compilation. Add the `view` visibility to resolve the warning

```solidity
File: contracts/AaveV3YieldSource.sol   #1

203    function balanceOfToken(address _user) external override returns (uint256) {
```
https://github.com/pooltogether/aave-v3-yield-source/blob/e63d1b0e396a5bce89f093630c282ca1c6627e44/contracts/AaveV3YieldSource.sol#L203

### 3. `public` functions not called by the contract should be declared `external` instead
Contracts [are allowed](https://docs.soliditylang.org/en/latest/contracts.html#function-overriding) to override their parents' functions and change the visibility from `external` to `public`.

```solidity
File: contracts/AaveV3YieldSource.sol   #1

211   function depositToken() public view override returns (address) {
```
https://github.com/pooltogether/aave-v3-yield-source/blob/e63d1b0e396a5bce89f093630c282ca1c6627e44/contracts/AaveV3YieldSource.sol#L211

### 4. Inconsistent variable-name-to-variable-type usage
In the case below `_token` is an `address` whereas in all other instances, `_token` is an `IERC20`. Changing the name of the variable to something like `_tokenAddr` will make the code more readable and consistent

```solidity
File: contracts/AaveV3YieldSource.sol   #1

348    function _requireNotAToken(address _token) internal view {
```
https://github.com/pooltogether/aave-v3-yield-source/blob/e63d1b0e396a5bce89f093630c282ca1c6627e44/contracts/AaveV3YieldSource.sol#L348

### 5. Typos

```solidity
File: contracts/AaveV3YieldSource.sol   #1

38      * @param decimals Number of decimals the shares (inhereted ERC20) will have. Same as underlying asset to ensure sane exchange rates for shares.
```
inhereted
https://github.com/pooltogether/aave-v3-yield-source/blob/e63d1b0e396a5bce89f093630c282ca1c6627e44/contracts/AaveV3YieldSource.sol#L38

```solidity
File: contracts/AaveV3YieldSource.sol   #2

156      * @param decimals_ Number of decimals the shares (inhereted ERC20) will have. Same as underlying asset to ensure sane exchange rates for shares.
```
inhereted
https://github.com/pooltogether/aave-v3-yield-source/blob/e63d1b0e396a5bce89f093630c282ca1c6627e44/contracts/AaveV3YieldSource.sol#L156

### 6. Grammar
A lot of the NatSpec/comments add a period to the end of fragments. Periods should only be used when there is both a noun phrase and a verb phrase

```solidity
File: contracts/AaveV3YieldSource.sol (various lines)   #1

```
https://github.com/pooltogether/aave-v3-yield-source/blob/e63d1b0e396a5bce89f093630c282ca1c6627e44/contracts/AaveV3YieldSource.sol

### 7. Use a more recent version of solidity
Use a solidity version of at least 0.8.13 to get the ability to use `using for` with a list of free functions

```solidity
File: contracts/AaveV3YieldSource.sol   #1

3   pragma solidity 0.8.10;
```
https://github.com/pooltogether/aave-v3-yield-source/blob/e63d1b0e396a5bce89f093630c282ca1c6627e44/contracts/AaveV3YieldSource.sol#L3

### 8. Function behavior doesn't match name
The line below should use `_requireNotAToken()` but it doesn't because that function's `revert()` string specifically refers to the 'allowance' functions. The function NatSpec doesn't mention this fact. If the function wants different strings based on where it's called from, it can use `msg.sig` to choose the right one. An even better approach would be to have a custom error instead of a revert string, and include the `msg.sig` as an argument to the error.

```solidity
File: contracts/AaveV3YieldSource.sol   #1

337      require(address(_token) != address(aToken), "AaveV3YS/forbid-aToken-transfer");
```
https://github.com/pooltogether/aave-v3-yield-source/blob/e63d1b0e396a5bce89f093630c282ca1c6627e44/contracts/AaveV3YieldSource.sol#L337

### 9. Unneeded functions
The `transferERC20()` function is sufficient for handling unexpected tokens; the increase/decrease allowance functions aren't useful. Approval isn't required for the contract itself to do the transfer when told to do it, but the increase function requires a second operation to actually do the transfer. Even if there is a case where funds can be moved by an existing contract's functionality, that contract might pass along its own token to this contract, starting another issue. The increase/decrease functions just add an extra attack surface and should just be removed.

```solidity
File: contracts/AaveV3YieldSource.sol   #1

315    function increaseERC20Allowance(
316      IERC20 _token,
317      address _spender,
318      uint256 _amount
319    ) external onlyManagerOrOwner {
320      _requireNotAToken(address(_token));
321      _token.safeIncreaseAllowance(_spender, _amount);
322      emit IncreasedERC20Allowance(msg.sender, _spender, _amount, _token);
323    }
324  
325    /**
326     * @notice Transfer ERC20 tokens other than the aTokens held by this contract to the recipient address.
327     * @dev This function is only callable by the owner or asset manager.
328     * @param _token Address of the ERC20 token to transfer
329     * @param _to Address of the recipient of the tokens
330     * @param _amount Amount of tokens to transfer
331     */
332    function transferERC20(
333      IERC20 _token,
334      address _to,
335      uint256 _amount
336    ) external onlyManagerOrOwner {
337      require(address(_token) != address(aToken), "AaveV3YS/forbid-aToken-transfer");
338      _token.safeTransfer(_to, _amount);
339      emit TransferredERC20(msg.sender, _to, _amount, _token);
340    }
```
https://github.com/pooltogether/aave-v3-yield-source/blob/e63d1b0e396a5bce89f093630c282ca1c6627e44/contracts/AaveV3YieldSource.sol#L315-L340

### 10. Natspec descriptions incorrect
The instances below say that the argument is an `address` but they're in fact all variables of type contract. Internally solidity translates contracts to addresses when passing them to `abi` calls and when emitting events, but the compiler requires the specific user-defined type and errors if a simple address is provided without a cast instead.

```solidity
File: contracts/AaveV3YieldSource.sol   #1

33     * @param aToken Aave aToken address
34     * @param rewardsController Aave rewardsController address
35     * @param poolAddressesProviderRegistry Aave poolAddressesProviderRegistry address
```
https://github.com/pooltogether/aave-v3-yield-source/blob/e63d1b0e396a5bce89f093630c282ca1c6627e44/contracts/AaveV3YieldSource.sol#L33-L35

```solidity
File: contracts/AaveV3YieldSource.sol   #2

87     * @param token Address of the ERC20 token to decrease allowance for
```
https://github.com/pooltogether/aave-v3-yield-source/blob/e63d1b0e396a5bce89f093630c282ca1c6627e44/contracts/AaveV3YieldSource.sol#L87

```solidity
File: contracts/AaveV3YieldSource.sol   #3

101     * @param token Address of the ERC20 token to increase allowance for
```
https://github.com/pooltogether/aave-v3-yield-source/blob/e63d1b0e396a5bce89f093630c282ca1c6627e44/contracts/AaveV3YieldSource.sol#L101

```solidity
File: contracts/AaveV3YieldSource.sol   #4

115     * @param token Address of the ERC20 token transferred
```
https://github.com/pooltogether/aave-v3-yield-source/blob/e63d1b0e396a5bce89f093630c282ca1c6627e44/contracts/AaveV3YieldSource.sol#L115

```solidity
File: contracts/AaveV3YieldSource.sol   #5

126    /// @notice Yield-bearing Aave aToken address.
```
https://github.com/pooltogether/aave-v3-yield-source/blob/e63d1b0e396a5bce89f093630c282ca1c6627e44/contracts/AaveV3YieldSource.sol#L126

```solidity
File: contracts/AaveV3YieldSource.sol   #6

129    /// @notice Aave RewardsController address.
```
https://github.com/pooltogether/aave-v3-yield-source/blob/e63d1b0e396a5bce89f093630c282ca1c6627e44/contracts/AaveV3YieldSource.sol#L129

```solidity
File: contracts/AaveV3YieldSource.sol   #7

132    /// @notice Aave poolAddressesProviderRegistry address.
```
https://github.com/pooltogether/aave-v3-yield-source/blob/e63d1b0e396a5bce89f093630c282ca1c6627e44/contracts/AaveV3YieldSource.sol#L132

```solidity
File: contracts/AaveV3YieldSource.sol   #8

151     * @param _aToken Aave aToken address
```
https://github.com/pooltogether/aave-v3-yield-source/blob/e63d1b0e396a5bce89f093630c282ca1c6627e44/contracts/AaveV3YieldSource.sol#L151

```solidity
File: contracts/AaveV3YieldSource.sol   #9

152     * @param _rewardsController Aave rewardsController address
```
https://github.com/pooltogether/aave-v3-yield-source/blob/e63d1b0e396a5bce89f093630c282ca1c6627e44/contracts/AaveV3YieldSource.sol#L152

### 11. Event is missing `indexed` fields
Each `event` should use three `indexed` fields if there are three or more fields

```solidity
File: contracts/AaveV3YieldSource.sol   #1

41     event AaveV3YieldSourceInitialized(
42       IAToken indexed aToken,
43       IRewardsController rewardsController,
44       IPoolAddressesProviderRegistry poolAddressesProviderRegistry,
45       string name,
46       string symbol,
47       uint8 decimals,
48       address owner
49     );
```
https://github.com/pooltogether/aave-v3-yield-source/blob/e63d1b0e396a5bce89f093630c282ca1c6627e44/contracts/AaveV3YieldSource.sol#L41-L49

```solidity
File: contracts/AaveV3YieldSource.sol   #2

58     event SuppliedTokenTo(address indexed from, uint256 shares, uint256 amount, address indexed to);
```
https://github.com/pooltogether/aave-v3-yield-source/blob/e63d1b0e396a5bce89f093630c282ca1c6627e44/contracts/AaveV3YieldSource.sol#L58

```solidity
File: contracts/AaveV3YieldSource.sol   #3

66     event RedeemedToken(address indexed from, uint256 shares, uint256 amount);
```
https://github.com/pooltogether/aave-v3-yield-source/blob/e63d1b0e396a5bce89f093630c282ca1c6627e44/contracts/AaveV3YieldSource.sol#L66

```solidity
File: contracts/AaveV3YieldSource.sol   #4

75     event Claimed(
76       address indexed from,
77       address indexed to,
78       address[] rewardsList,
79       uint256[] claimedAmounts
80     );
```
https://github.com/pooltogether/aave-v3-yield-source/blob/e63d1b0e396a5bce89f093630c282ca1c6627e44/contracts/AaveV3YieldSource.sol#L75-L80

### 12. Non-exploitable re-entrancies
Code should follow the best-practice of [check-effects-interaction](https://blockchain-academy.hs-mittweida.de/courses/solidity-coding-beginners-to-intermediate/lessons/solidity-11-coding-patterns/topic/checks-effects-interactions/)

```
Reentrancy in AaveV3YieldSource.supplyTokenTo(uint256,address) (contracts/AaveV3YieldSource.sol#231-242):  #1
    External calls:
    - IERC20(_underlyingAssetAddress).safeTransferFrom(msg.sender,address(this),_depositAmount) (contracts/AaveV3YieldSource.sol#236)
    - _pool().supply(_underlyingAssetAddress,_depositAmount,address(this),REFERRAL_CODE) (contracts/AaveV3YieldSource.sol#237)
    State variables written after the call(s):
    - _mint(_to,_shares) (contracts/AaveV3YieldSource.sol#239)
        - _totalSupply += amount (node_modules/@openzeppelin/contracts/token/ERC20/ERC20.sol#262)
```

```
Reentrancy in AaveV3YieldSource.supplyTokenTo(uint256,address) (contracts/AaveV3YieldSource.sol#231-242):  #2
    External calls:
    - IERC20(_underlyingAssetAddress).safeTransferFrom(msg.sender,address(this),_depositAmount) (contracts/AaveV3YieldSource.sol#236)
    - _pool().supply(_underlyingAssetAddress,_depositAmount,address(this),REFERRAL_CODE) (contracts/AaveV3YieldSource.sol#237)
    State variables written after the call(s):
    - _mint(_to,_shares) (contracts/AaveV3YieldSource.sol#239)
        - _balances[account] += amount (node_modules/@openzeppelin/contracts/token/ERC20/ERC20.sol#263)
```

```
Reentrancy in AaveV3YieldSource.claimRewards(address) (contracts/AaveV3YieldSource.sol#275-286):  #3
    External calls:
    - (_rewardsList,_claimedAmounts) = rewardsController.claimAllRewards(_assets,_to) (contracts/AaveV3YieldSource.sol#281-282)
    Event emitted after the call(s):
    - Claimed(msg.sender,_to,_rewardsList,_claimedAmounts) (contracts/AaveV3YieldSource.sol#284)
```

```
Reentrancy in AaveV3YieldSource.decreaseERC20Allowance(IERC20,address,uint256) (contracts/AaveV3YieldSource.sol#296-304):  #4
    External calls:
    - _token.safeDecreaseAllowance(_spender,_amount) (contracts/AaveV3YieldSource.sol#302)
    Event emitted after the call(s):
    - DecreasedERC20Allowance(msg.sender,_spender,_amount,_token) (contracts/AaveV3YieldSource.sol#303)
```

```
Reentrancy in AaveV3YieldSource.increaseERC20Allowance(IERC20,address,uint256) (contracts/AaveV3YieldSource.sol#315-323):  #5
    External calls:
    - _token.safeIncreaseAllowance(_spender,_amount) (contracts/AaveV3YieldSource.sol#321)
    Event emitted after the call(s):
    - IncreasedERC20Allowance(msg.sender,_spender,_amount,_token) (contracts/AaveV3YieldSource.sol#322)
```

```
Reentrancy in AaveV3YieldSource.redeemToken(uint256) (contracts/AaveV3YieldSource.sol#251-267):  #6
    External calls:
    - _pool().withdraw(_underlyingAssetAddress,_redeemAmount,address(this)) (contracts/AaveV3YieldSource.sol#259)
    - _assetToken.safeTransfer(msg.sender,_balanceDiff) (contracts/AaveV3YieldSource.sol#263)
    Event emitted after the call(s):
    - RedeemedToken(msg.sender,_shares,_redeemAmount) (contracts/AaveV3YieldSource.sol#265)
```

```
Reentrancy in AaveV3YieldSource.supplyTokenTo(uint256,address) (contracts/AaveV3YieldSource.sol#231-242):  #7
    External calls:
    - IERC20(_underlyingAssetAddress).safeTransferFrom(msg.sender,address(this),_depositAmount) (contracts/AaveV3YieldSource.sol#236)
    - _pool().supply(_underlyingAssetAddress,_depositAmount,address(this),REFERRAL_CODE) (contracts/AaveV3YieldSource.sol#237)
    Event emitted after the call(s):
    - SuppliedTokenTo(msg.sender,_shares,_depositAmount,_to) (contracts/AaveV3YieldSource.sol#241)
    - Transfer(address(0),account,amount) (node_modules/@openzeppelin/contracts/token/ERC20/ERC20.sol#264)
        - _mint(_to,_shares) (contracts/AaveV3YieldSource.sol#239)
```

```
Reentrancy in AaveV3YieldSource.transferERC20(IERC20,address,uint256) (contracts/AaveV3YieldSource.sol#332-340):  #8
    External calls:
    - _token.safeTransfer(_to,_amount) (contracts/AaveV3YieldSource.sol#338)
    Event emitted after the call(s):
    - TransferredERC20(msg.sender,_to,_amount,_token) (contracts/AaveV3YieldSource.sol#339)
```