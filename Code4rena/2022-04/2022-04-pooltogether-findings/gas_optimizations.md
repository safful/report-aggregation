## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Gas Optimizations](https://github.com/code-423n4/2022-04-pooltogether-findings/issues/84) 

### 1. State variables only set in the constructor should be declared `immutable`
Avoids a Gsset (20000 gas) in the constructor, and replaces each Gwarmacces (100 gas) with a `PUSH32` (3 gas)

```solidity
File: contracts/AaveV3YieldSource.sol   #1

127     IAToken public aToken;
```
https://github.com/pooltogether/aave-v3-yield-source/blob/e63d1b0e396a5bce89f093630c282ca1c6627e44/contracts/AaveV3YieldSource.sol#L127

```solidity
File: contracts/AaveV3YieldSource.sol   #2

130     IRewardsController public rewardsController;
```
https://github.com/pooltogether/aave-v3-yield-source/blob/e63d1b0e396a5bce89f093630c282ca1c6627e44/contracts/AaveV3YieldSource.sol#L130

```solidity
File: contracts/AaveV3YieldSource.sol   #3

133     IPoolAddressesProviderRegistry public poolAddressesProviderRegistry;
```
https://github.com/pooltogether/aave-v3-yield-source/blob/e63d1b0e396a5bce89f093630c282ca1c6627e44/contracts/AaveV3YieldSource.sol#L133


```diff
diff --git a/AaveV3YieldSource.sol.orig b/AaveV3YieldSource.sol.new
index 3975311..1d229ff 100644
--- a/AaveV3YieldSource.sol.orig
+++ b/AaveV3YieldSource.sol.new
@@ -124,13 +124,13 @@ contract AaveV3YieldSource is ERC20, IYieldSource, Manageable, ReentrancyGuard {
   /* ============ Variables ============ */
 
   /// @notice Yield-bearing Aave aToken address.
-  IAToken public aToken;
+  IAToken public immutable aToken;
 
   /// @notice Aave RewardsController address.
-  IRewardsController public rewardsController;
+  IRewardsController public immutable rewardsController;
 
   /// @notice Aave poolAddressesProviderRegistry address.
-  IPoolAddressesProviderRegistry public poolAddressesProviderRegistry;
+  IPoolAddressesProviderRegistry public immutable poolAddressesProviderRegistry;
 
   /// @notice ERC20 token decimals.
   uint8 private immutable _decimals;
```

```diff
diff --git a/gas.orig b/gas.new
index d87edc2..a6cd51d 100644
--- a/gas.orig
+++ b/gas.new
@@ -5,21 +5,21 @@
 ·····························|··························|·············|·············|·············|···············|··············
 |  Contract                  ·  Method                  ·  Min        ·  Max        ·  Avg        ·  # calls      ·  usd (avg)  │
 ·····························|··························|·············|·············|·············|···············|··············
-|  AaveV3YieldSourceHarness  ·  claimRewards            ·      54735  ·      56896  ·      55816  ·            4  ·          -  │
+|  AaveV3YieldSourceHarness  ·  claimRewards            ·      50521  ·      52682  ·      51602  ·            4  ·          -  │
 ·····························|··························|·············|·············|·············|···············|··············
-|  AaveV3YieldSourceHarness  ·  decreaseERC20Allowance  ·      37465  ·      39638  ·      38910  ·            3  ·          -  │
+|  AaveV3YieldSourceHarness  ·  decreaseERC20Allowance  ·      35374  ·      37547  ·      36819  ·            3  ·          -  │
 ·····························|··························|·············|·············|·············|···············|··············
-|  AaveV3YieldSourceHarness  ·  increaseERC20Allowance  ·      59133  ·      61618  ·      61212  ·            7  ·          -  │
+|  AaveV3YieldSourceHarness  ·  increaseERC20Allowance  ·      57042  ·      59527  ·      59121  ·            7  ·          -  │
 ·····························|··························|·············|·············|·············|···············|··············
 |  AaveV3YieldSourceHarness  ·  mint                    ·      51381  ·      68493  ·      61904  ·           13  ·          -  │
 ·····························|··························|·············|·············|·············|···············|··············
-|  AaveV3YieldSourceHarness  ·  redeemToken             ·          -  ·          -  ·     110713  ·            1  ·          -  │
+|  AaveV3YieldSourceHarness  ·  redeemToken             ·          -  ·          -  ·     106359  ·            1  ·          -  │
 ·····························|··························|·············|·············|·············|···············|··············
 |  AaveV3YieldSourceHarness  ·  setManager              ·          -  ·          -  ·      47982  ·            4  ·          -  │
 ·····························|··························|·············|·············|·············|···············|··············
-|  AaveV3YieldSourceHarness  ·  supplyTokenTo           ·     110668  ·     153720  ·     142217  ·            6  ·          -  │
+|  AaveV3YieldSourceHarness  ·  supplyTokenTo           ·     106314  ·     149466  ·     137930  ·            6  ·          -  │
 ·····························|··························|·············|·············|·············|···············|··············
-|  AaveV3YieldSourceHarness  ·  transferERC20           ·      58318  ·      60467  ·      59393  ·            2  ·          -  │
+|  AaveV3YieldSourceHarness  ·  transferERC20           ·      56227  ·      58376  ·      57302  ·            2  ·          -  │
 ·····························|··························|·············|·············|·············|···············|··············
 |  ERC20Mintable             ·  approve                 ·      26692  ·      46592  ·      43275  ·            6  ·          -  │
 ·····························|··························|·············|·············|·············|···············|··············
@@ -29,7 +29,7 @@
 ·····························|··························|·············|·············|·············|···············|··············
 |  Deployments                                          ·                                         ·  % of limit   ·             │
 ························································|·············|·············|·············|···············|··············
-|  AaveV3YieldSourceHarness                             ·    2505760  ·    2505772  ·    2505770  ·        8.4 %  ·          -  │
+|  AaveV3YieldSourceHarness                             ·    2497750  ·    2497762  ·    2497760  ·        8.3 %  ·          -  │
 ························································|·············|·············|·············|···············|··············
 |  ERC20Mintable                                        ·          -  ·          -  ·    1154099  ·        3.8 %  ·          -  │
 ·-------------------------------------------------------|-------------|-------------|-------------|---------------|-------------·
```


### 2. `internal` functions only called once can be inlined to save gas
Not inlining costs 20 to 40 gas because of two extra `JUMP` instructions and additional stack operations needed for function calls.

```solidity
File: contracts/AaveV3YieldSource.sol   #1

369     function _sharesToToken(uint256 _shares) internal view returns (uint256) {
```
https://github.com/pooltogether/aave-v3-yield-source/blob/e63d1b0e396a5bce89f093630c282ca1c6627e44/contracts/AaveV3YieldSource.sol#L369

```solidity
File: contracts/AaveV3YieldSource.sol   #2

388     function _poolProvider() internal view returns (IPoolAddressesProvider) {
```
https://github.com/pooltogether/aave-v3-yield-source/blob/e63d1b0e396a5bce89f093630c282ca1c6627e44/contracts/AaveV3YieldSource.sol#L388

```diff
diff --git a/AaveV3YieldSource.sol.orig b/AaveV3YieldSource.sol.new
index 3975311..d7ec7ec 100644
--- a/AaveV3YieldSource.sol.orig
+++ b/AaveV3YieldSource.sol.new
@@ -201,7 +201,12 @@ contract AaveV3YieldSource is ERC20, IYieldSource, Manageable, ReentrancyGuard {
    * @return The underlying balance of asset tokens.
    */
   function balanceOfToken(address _user) external override returns (uint256) {
-    return _sharesToToken(balanceOf(_user));
+    uint256 _shares = balanceOf(_user);
+    uint256 _supply = totalSupply();
+
+    // tokens = (shares * yieldSourceATokenTotalSupply) / totalShares
+    return _supply == 0 ? _shares : _shares.mul(aToken.balanceOf(address(this))).div(_supply);
+ 
   }
 
   /**
@@ -361,18 +366,6 @@ contract AaveV3YieldSource is ERC20, IYieldSource, Manageable, ReentrancyGuard {
     return _supply == 0 ? _tokens : _tokens.mul(_supply).div(aToken.balanceOf(address(this)));
   }
 
-  /**
-   * @notice Calculates the number of asset tokens a user has in the yield source.
-   * @param _shares Amount of shares
-   * @return Number of asset tokens.
-   */
-  function _sharesToToken(uint256 _shares) internal view returns (uint256) {
-    uint256 _supply = totalSupply();
-
-    // tokens = (shares * yieldSourceATokenTotalSupply) / totalShares
-    return _supply == 0 ? _shares : _shares.mul(aToken.balanceOf(address(this))).div(_supply);
-  }
-
   /**
    * @notice Returns the underlying asset token address.
    * @return Underlying asset token address.
@@ -381,22 +374,13 @@ contract AaveV3YieldSource is ERC20, IYieldSource, Manageable, ReentrancyGuard {
     return aToken.UNDERLYING_ASSET_ADDRESS();
   }
 
-  /**
-   * @notice Retrieves Aave PoolAddressesProvider address.
-   * @return A reference to PoolAddressesProvider interface.
-   */
-  function _poolProvider() internal view returns (IPoolAddressesProvider) {
-    return
-      IPoolAddressesProvider(
-        poolAddressesProviderRegistry.getAddressesProvidersList()[ADDRESSES_PROVIDER_ID]
-      );
-  }
-
   /**
    * @notice Retrieves Aave Pool address.
    * @return A reference to Pool interface.
    */
   function _pool() internal view returns (IPool) {
-    return IPool(_poolProvider().getPool());
+    return IPool(IPoolAddressesProvider(
+        poolAddressesProviderRegistry.getAddressesProvidersList()[ADDRESSES_PROVIDER_ID]
+      ).getPool());
   }
 }
```

```diff
diff --git a/gas.orig b/gas.new
index d87edc2..6948266 100644
--- a/gas.orig
+++ b/gas.new
@@ -13,11 +13,11 @@
 ·····························|··························|·············|·············|·············|···············|··············
 |  AaveV3YieldSourceHarness  ·  mint                    ·      51381  ·      68493  ·      61904  ·           13  ·          -  │
 ·····························|··························|·············|·············|·············|···············|··············
-|  AaveV3YieldSourceHarness  ·  redeemToken             ·          -  ·          -  ·     110713  ·            1  ·          -  │
+|  AaveV3YieldSourceHarness  ·  redeemToken             ·          -  ·          -  ·     110678  ·            1  ·          -  │
 ·····························|··························|·············|·············|·············|···············|··············
 |  AaveV3YieldSourceHarness  ·  setManager              ·          -  ·          -  ·      47982  ·            4  ·          -  │
 ·····························|··························|·············|·············|·············|···············|··············
-|  AaveV3YieldSourceHarness  ·  supplyTokenTo           ·     110668  ·     153720  ·     142217  ·            6  ·          -  │
+|  AaveV3YieldSourceHarness  ·  supplyTokenTo           ·     110633  ·     153685  ·     142182  ·            6  ·          -  │
 ·····························|··························|·············|·············|·············|···············|··············
 |  AaveV3YieldSourceHarness  ·  transferERC20           ·      58318  ·      60467  ·      59393  ·            2  ·          -  │
 ·····························|··························|·············|·············|·············|···············|··············
@@ -29,7 +29,7 @@
 ·····························|··························|·············|·············|·············|···············|··············
 |  Deployments                                          ·                                         ·  % of limit   ·             │
 ························································|·············|·············|·············|···············|··············
-|  AaveV3YieldSourceHarness                             ·    2505760  ·    2505772  ·    2505770  ·        8.4 %  ·          -  │
+|  AaveV3YieldSourceHarness                             ·    2559043  ·    2559055  ·    2559053  ·        8.5 %  ·          -  │
 ························································|·············|·············|·············|···············|··············
 |  ERC20Mintable                                        ·          -  ·          -  ·    1154099  ·        3.8 %  ·          -  │
 ·-------------------------------------------------------|-------------|-------------|-------------|---------------|-------------·
```
### 3. Using `> 0` costs more gas than `!= 0` when used on a `uint` in a `require()` statement
This change saves [6 gas](https://aws1.discourse-cdn.com/business6/uploads/zeppelin/original/2X/3/363a367d6d68851f27d2679d10706cd16d788b96.png) per instance

```solidity
File: contracts/AaveV3YieldSource.sol   #1

179       require(decimals_ > 0, "AaveV3YS/decimals-gt-zero");
```
https://github.com/pooltogether/aave-v3-yield-source/blob/e63d1b0e396a5bce89f093630c282ca1c6627e44/contracts/AaveV3YieldSource.sol#L179

```solidity
File: contracts/AaveV3YieldSource.sol   #2

233       require(_shares > 0, "AaveV3YS/shares-gt-zero");
```
https://github.com/pooltogether/aave-v3-yield-source/blob/e63d1b0e396a5bce89f093630c282ca1c6627e44/contracts/AaveV3YieldSource.sol#L233

```diff
diff --git a/AaveV3YieldSource.sol.orig b/AaveV3YieldSource.sol.new
index 3975311..e00ad47 100644
--- a/AaveV3YieldSource.sol.orig
+++ b/AaveV3YieldSource.sol.new
@@ -176,7 +176,7 @@ contract AaveV3YieldSource is ERC20, IYieldSource, Manageable, ReentrancyGuard {
 
     require(_owner != address(0), "AaveV3YS/owner-not-zero-address");
 
-    require(decimals_ > 0, "AaveV3YS/decimals-gt-zero");
+    require(decimals_ != 0, "AaveV3YS/decimals-gt-zero");
     _decimals = decimals_;
 
     // Approve once for max amount
@@ -230,7 +230,7 @@ contract AaveV3YieldSource is ERC20, IYieldSource, Manageable, ReentrancyGuard {
    */
   function supplyTokenTo(uint256 _depositAmount, address _to) external override nonReentrant {
     uint256 _shares = _tokenToShares(_depositAmount);
-    require(_shares > 0, "AaveV3YS/shares-gt-zero");
+    require(_shares != 0, "AaveV3YS/shares-gt-zero");
 
     address _underlyingAssetAddress = _tokenAddress();
     IERC20(_underlyingAssetAddress).safeTransferFrom(msg.sender, address(this), _depositAmount);
```

```diff
diff --git a/gas.orig b/gas.new
index d87edc2..9a90ad3 100644
--- a/gas.orig
+++ b/gas.new
@@ -17,7 +17,7 @@
 ·····························|··························|·············|·············|·············|···············|··············
 |  AaveV3YieldSourceHarness  ·  setManager              ·          -  ·          -  ·      47982  ·            4  ·          -  │
 ·····························|··························|·············|·············|·············|···············|··············
-|  AaveV3YieldSourceHarness  ·  supplyTokenTo           ·     110668  ·     153720  ·     142217  ·            6  ·          -  │
+|  AaveV3YieldSourceHarness  ·  supplyTokenTo           ·     110662  ·     153714  ·     142211  ·            6  ·          -  │
 ·····························|··························|·············|·············|·············|···············|··············
 |  AaveV3YieldSourceHarness  ·  transferERC20           ·      58318  ·      60467  ·      59393  ·            2  ·          -  │
 ·····························|··························|·············|·············|·············|···············|··············
@@ -29,7 +29,7 @@
 ·····························|··························|·············|·············|·············|···············|··············
 |  Deployments                                          ·                                         ·  % of limit   ·             │
 ························································|·············|·············|·············|···············|··············
-|  AaveV3YieldSourceHarness                             ·    2505760  ·    2505772  ·    2505770  ·        8.4 %  ·          -  │
+|  AaveV3YieldSourceHarness                             ·    2505106  ·    2505118  ·    2505116  ·        8.4 %  ·          -  │
 ························································|·············|·············|·············|···············|··············
 |  ERC20Mintable                                        ·          -  ·          -  ·    1154099  ·        3.8 %  ·          -  │
 ·-------------------------------------------------------|-------------|-------------|-------------|---------------|-------------·
```

### 4. Usage of `uints`/`ints` smaller than 32 bytes (256 bits) incurs overhead
> When using elements that are smaller than 32 bytes, your contract’s gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.

https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html
Use a larger size then downcast where needed

```solidity
File: contracts/AaveV3YieldSource.sol   #1

47       uint8 decimals,
```
https://github.com/pooltogether/aave-v3-yield-source/blob/e63d1b0e396a5bce89f093630c282ca1c6627e44/contracts/AaveV3YieldSource.sol#L47

```solidity
File: contracts/AaveV3YieldSource.sol   #2

136     uint8 private immutable _decimals;
```
https://github.com/pooltogether/aave-v3-yield-source/blob/e63d1b0e396a5bce89f093630c282ca1c6627e44/contracts/AaveV3YieldSource.sol#L136

```solidity
File: contracts/AaveV3YieldSource.sol   #3

145     uint16 private constant REFERRAL_CODE = uint16(188);
```
https://github.com/pooltogether/aave-v3-yield-source/blob/e63d1b0e396a5bce89f093630c282ca1c6627e44/contracts/AaveV3YieldSource.sol#L145

```solidity
File: contracts/AaveV3YieldSource.sol   #4

165       uint8 decimals_,
```
https://github.com/pooltogether/aave-v3-yield-source/blob/e63d1b0e396a5bce89f093630c282ca1c6627e44/contracts/AaveV3YieldSource.sol#L165

```diff
diff --git a/AaveV3YieldSource.sol.orig b/AaveV3YieldSource.sol.new
index 3975311..0dfa477 100644
--- a/AaveV3YieldSource.sol.orig
+++ b/AaveV3YieldSource.sol.new
@@ -44,7 +44,7 @@ contract AaveV3YieldSource is ERC20, IYieldSource, Manageable, ReentrancyGuard {
     IPoolAddressesProviderRegistry poolAddressesProviderRegistry,
     string name,
     string symbol,
-    uint8 decimals,
+    uint256 decimals,
     address owner
   );
 
@@ -133,7 +133,7 @@ contract AaveV3YieldSource is ERC20, IYieldSource, Manageable, ReentrancyGuard {
   IPoolAddressesProviderRegistry public poolAddressesProviderRegistry;
 
   /// @notice ERC20 token decimals.
-  uint8 private immutable _decimals;
+  uint256 private immutable _decimals;
 
   /**
    * @dev Aave genesis market PoolAddressesProvider's ID.
@@ -142,7 +142,7 @@ contract AaveV3YieldSource is ERC20, IYieldSource, Manageable, ReentrancyGuard {
   uint256 private constant ADDRESSES_PROVIDER_ID = uint256(0);
 
   /// @dev PoolTogether's Aave Referral Code
-  uint16 private constant REFERRAL_CODE = uint16(188);
+  uint256 private constant REFERRAL_CODE = 188;
 
   /* ============ Constructor ============ */
 
@@ -162,7 +162,7 @@ contract AaveV3YieldSource is ERC20, IYieldSource, Manageable, ReentrancyGuard {
     IPoolAddressesProviderRegistry _poolAddressesProviderRegistry,
     string memory _name,
     string memory _symbol,
-    uint8 decimals_,
+    uint256 decimals_,
     address _owner
   ) Ownable(_owner) ERC20(_name, _symbol) ReentrancyGuard() {
     require(address(_aToken) != address(0), "AaveV3YS/aToken-not-zero-address");
@@ -218,7 +218,7 @@ contract AaveV3YieldSource is ERC20, IYieldSource, Manageable, ReentrancyGuard {
    * @return The number of decimals.
    */
   function decimals() public view virtual override returns (uint8) {
-    return _decimals;
+    return uint8(_decimals);
   }
 
   /**
@@ -234,7 +234,7 @@ contract AaveV3YieldSource is ERC20, IYieldSource, Manageable, ReentrancyGuard {
 
     address _underlyingAssetAddress = _tokenAddress();
     IERC20(_underlyingAssetAddress).safeTransferFrom(msg.sender, address(this), _depositAmount);
-    _pool().supply(_underlyingAssetAddress, _depositAmount, address(this), REFERRAL_CODE);
+    _pool().supply(_underlyingAssetAddress, _depositAmount, address(this), uint16(REFERRAL_CODE));
 
     _mint(_to, _shares);
```

```diff
diff --git a/gas.orig b/gas.new
index d87edc2..367daee 100644
--- a/gas.orig
+++ b/gas.new
@@ -29,7 +29,7 @@
 ·····························|··························|·············|·············|·············|···············|··············
 |  Deployments                                          ·                                         ·  % of limit   ·             │
 ························································|·············|·············|·············|···············|··············
-|  AaveV3YieldSourceHarness                             ·    2505760  ·    2505772  ·    2505770  ·        8.4 %  ·          -  │
+|  AaveV3YieldSourceHarness                             ·    2505659  ·    2505671  ·    2505669  ·        8.4 %  ·          -  │
 ························································|·············|·············|·············|···············|··············
 |  ERC20Mintable                                        ·          -  ·          -  ·    1154099  ·        3.8 %  ·          -  │
 ·-------------------------------------------------------|-------------|-------------|-------------|---------------|-------------·
```

### 5. Don't use `SafeMath` once the solidity version is 0.8.0 or greater
Version 0.8.0 introduces internal overflow checks, so using `SafeMath` is redundant and adds overhead

```solidity
File: contracts/AaveV3YieldSource.sol   #1

262   uint256 _balanceDiff = _afterBalance.sub(_beforeBalance);
```
https://github.com/pooltogether/aave-v3-yield-source/blob/e63d1b0e396a5bce89f093630c282ca1c6627e44/contracts/AaveV3YieldSource.sol#L262

```solidity
File: contracts/AaveV3YieldSource.sol   #2

361   return _supply == 0 ? _tokens : _tokens.mul(_supply).div(aToken.balanceOf(address(this)));
```
https://github.com/pooltogether/aave-v3-yield-source/blob/e63d1b0e396a5bce89f093630c282ca1c6627e44/contracts/AaveV3YieldSource.sol#L361

```solidity
File: contracts/AaveV3YieldSource.sol   #3

373   return _supply == 0 ? _shares : _shares.mul(aToken.balanceOf(address(this))).div(_supply);
```
https://github.com/pooltogether/aave-v3-yield-source/blob/e63d1b0e396a5bce89f093630c282ca1c6627e44/contracts/AaveV3YieldSource.sol#L373

```diff
diff --git a/AaveV3YieldSource.sol.orig b/AaveV3YieldSource.sol.new
index 3975311..6346344 100644
--- a/AaveV3YieldSource.sol.orig
+++ b/AaveV3YieldSource.sol.new
@@ -11,7 +11,6 @@ import { IRewardsController } from "@aave/periphery-v3/contracts/rewards/interfa
 import { ERC20 } from "@openzeppelin/contracts/token/ERC20/ERC20.sol";
 import { IERC20 } from "@openzeppelin/contracts/token/ERC20/IERC20.sol";
 import { SafeERC20 } from "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";
-import { SafeMath } from "@openzeppelin/contracts/utils/math/SafeMath.sol";
 import { ReentrancyGuard } from "@openzeppelin/contracts/security/ReentrancyGuard.sol";
 
 import { Manageable, Ownable } from "@pooltogether/owner-manager-contracts/contracts/Manageable.sol";
@@ -23,7 +22,6 @@ import { IYieldSource } from "@pooltogether/yield-source-interface/contracts/IYi
  * @notice Yield Source for a PoolTogether prize pool that generates yield by depositing into Aave V3.
  */
 contract AaveV3YieldSource is ERC20, IYieldSource, Manageable, ReentrancyGuard {
-  using SafeMath for uint256;
   using SafeERC20 for IERC20;
 
   /* ============ Events ============ */
@@ -259,7 +257,7 @@ contract AaveV3YieldSource is ERC20, IYieldSource, Manageable, ReentrancyGuard {
     _pool().withdraw(_underlyingAssetAddress, _redeemAmount, address(this));
     uint256 _afterBalance = _assetToken.balanceOf(address(this));
 
-    uint256 _balanceDiff = _afterBalance.sub(_beforeBalance);
+    uint256 _balanceDiff = _afterBalance - _beforeBalance;
     _assetToken.safeTransfer(msg.sender, _balanceDiff);
 
     emit RedeemedToken(msg.sender, _shares, _redeemAmount);
@@ -358,7 +356,7 @@ contract AaveV3YieldSource is ERC20, IYieldSource, Manageable, ReentrancyGuard {
     uint256 _supply = totalSupply();
 
     // shares = (tokens * totalShares) / yieldSourceATokenTotalSupply
-    return _supply == 0 ? _tokens : _tokens.mul(_supply).div(aToken.balanceOf(address(this)));
+    return _supply == 0 ? _tokens : _tokens * _supply / aToken.balanceOf(address(this));
   }
 
   /**
@@ -370,7 +368,7 @@ contract AaveV3YieldSource is ERC20, IYieldSource, Manageable, ReentrancyGuard {
     uint256 _supply = totalSupply();
 
     // tokens = (shares * yieldSourceATokenTotalSupply) / totalShares
-    return _supply == 0 ? _shares : _shares.mul(aToken.balanceOf(address(this))).div(_supply);
+    return _supply == 0 ? _shares : _shares * aToken.balanceOf(address(this)) / _supply;
   }
 
   /**
```

```diff
diff --git a/gas.orig b/gas.new
index d87edc2..1c91b72 100644
--- a/gas.orig
+++ b/gas.new
@@ -13,11 +13,11 @@
 ·····························|··························|·············|·············|·············|···············|··············
 |  AaveV3YieldSourceHarness  ·  mint                    ·      51381  ·      68493  ·      61904  ·           13  ·          -  │
 ·····························|··························|·············|·············|·············|···············|··············
-|  AaveV3YieldSourceHarness  ·  redeemToken             ·          -  ·          -  ·     110713  ·            1  ·          -  │
+|  AaveV3YieldSourceHarness  ·  redeemToken             ·          -  ·          -  ·     110584  ·            1  ·          -  │
 ·····························|··························|·············|·············|·············|···············|··············
 |  AaveV3YieldSourceHarness  ·  setManager              ·          -  ·          -  ·      47982  ·            4  ·          -  │
 ·····························|··························|·············|·············|·············|···············|··············
-|  AaveV3YieldSourceHarness  ·  supplyTokenTo           ·     110668  ·     153720  ·     142217  ·            6  ·          -  │
+|  AaveV3YieldSourceHarness  ·  supplyTokenTo           ·     110584  ·     153720  ·     142189  ·            6  ·          -  │
 ·····························|··························|·············|·············|·············|···············|··············
 |  AaveV3YieldSourceHarness  ·  transferERC20           ·      58318  ·      60467  ·      59393  ·            2  ·          -  │
 ·····························|··························|·············|·············|·············|···············|··············
@@ -29,7 +29,7 @@
 ·····························|··························|·············|·············|·············|···············|··············
 |  Deployments                                          ·                                         ·  % of limit   ·             │
 ························································|·············|·············|·············|···············|··············
-|  AaveV3YieldSourceHarness                             ·    2505760  ·    2505772  ·    2505770  ·        8.4 %  ·          -  │
+|  AaveV3YieldSourceHarness                             ·    2497329  ·    2497341  ·    2497339  ·        8.3 %  ·          -  │
 ························································|·············|·············|·············|···············|··············
 |  ERC20Mintable                                        ·          -  ·          -  ·    1154099  ·        3.8 %  ·          -  │
 ·-------------------------------------------------------|-------------|-------------|-------------|---------------|-------------·
```

### 6. `require()` or `revert()` statements that check input arguments should be at the top of the function
Checks that involve constants should come before checks that involve state variables

```solidity
File: contracts/AaveV3YieldSource.sol   #1

177       require(_owner != address(0), "AaveV3YS/owner-not-zero-address");
```
https://github.com/pooltogether/aave-v3-yield-source/blob/e63d1b0e396a5bce89f093630c282ca1c6627e44/contracts/AaveV3YieldSource.sol#L177

```solidity
File: contracts/AaveV3YieldSource.sol   #2

179       require(decimals_ > 0, "AaveV3YS/decimals-gt-zero");
```
https://github.com/pooltogether/aave-v3-yield-source/blob/e63d1b0e396a5bce89f093630c282ca1c6627e44/contracts/AaveV3YieldSource.sol#L179

```diff
diff --git a/AaveV3YieldSource.sol.orig b/AaveV3YieldSource.sol.new
index 3975311..5d7d9fe 100644
--- a/AaveV3YieldSource.sol.orig
+++ b/AaveV3YieldSource.sol.new
@@ -166,17 +166,16 @@ contract AaveV3YieldSource is ERC20, IYieldSource, Manageable, ReentrancyGuard {
     address _owner
   ) Ownable(_owner) ERC20(_name, _symbol) ReentrancyGuard() {
     require(address(_aToken) != address(0), "AaveV3YS/aToken-not-zero-address");
+    require(_owner != address(0), "AaveV3YS/owner-not-zero-address");
+    require(decimals_ > 0, "AaveV3YS/decimals-gt-zero");
+    require(address(_rewardsController) != address(0), "AaveV3YS/RC-not-zero-address");
+    require(address(_poolAddressesProviderRegistry) != address(0), "AaveV3YS/PR-not-zero-address");
     aToken = _aToken;
 
-    require(address(_rewardsController) != address(0), "AaveV3YS/RC-not-zero-address");
     rewardsController = _rewardsController;
 
-    require(address(_poolAddressesProviderRegistry) != address(0), "AaveV3YS/PR-not-zero-address");
     poolAddressesProviderRegistry = _poolAddressesProviderRegistry;
 
-    require(_owner != address(0), "AaveV3YS/owner-not-zero-address");
-
-    require(decimals_ > 0, "AaveV3YS/decimals-gt-zero");
     _decimals = decimals_;
 
     // Approve once for max amount
```

```diff
diff --git a/gas.orig b/gas.new
index d87edc2..e76a03d 100644
--- a/gas.orig
+++ b/gas.new
@@ -29,7 +29,7 @@
 ·····························|··························|·············|·············|·············|···············|··············
 |  Deployments                                          ·                                         ·  % of limit   ·             │
 ························································|·············|·············|·············|···············|··············
-|  AaveV3YieldSourceHarness                             ·    2505760  ·    2505772  ·    2505770  ·        8.4 %  ·          -  │
+|  AaveV3YieldSourceHarness                             ·    2505548  ·    2505560  ·    2505558  ·        8.4 %  ·          -  │
 ························································|·············|·············|·············|···············|··············
 |  ERC20Mintable                                        ·          -  ·          -  ·    1154099  ·        3.8 %  ·          -  │
 ·-------------------------------------------------------|-------------|-------------|-------------|---------------|-------------·
```

### 7. Use custom errors rather than `revert()`/`require()` strings to save gas

```solidity
File: contracts/AaveV3YieldSource.sol   #1

168       require(address(_aToken) != address(0), "AaveV3YS/aToken-not-zero-address");
```
https://github.com/pooltogether/aave-v3-yield-source/blob/e63d1b0e396a5bce89f093630c282ca1c6627e44/contracts/AaveV3YieldSource.sol#L168

```solidity
File: contracts/AaveV3YieldSource.sol   #2

171       require(address(_rewardsController) != address(0), "AaveV3YS/RC-not-zero-address");
```
https://github.com/pooltogether/aave-v3-yield-source/blob/e63d1b0e396a5bce89f093630c282ca1c6627e44/contracts/AaveV3YieldSource.sol#L171

```solidity
File: contracts/AaveV3YieldSource.sol   #3

174       require(address(_poolAddressesProviderRegistry) != address(0), "AaveV3YS/PR-not-zero-address");
```
https://github.com/pooltogether/aave-v3-yield-source/blob/e63d1b0e396a5bce89f093630c282ca1c6627e44/contracts/AaveV3YieldSource.sol#L174

```solidity
File: contracts/AaveV3YieldSource.sol   #4

177       require(_owner != address(0), "AaveV3YS/owner-not-zero-address");
```
https://github.com/pooltogether/aave-v3-yield-source/blob/e63d1b0e396a5bce89f093630c282ca1c6627e44/contracts/AaveV3YieldSource.sol#L177

```solidity
File: contracts/AaveV3YieldSource.sol   #5

179       require(decimals_ > 0, "AaveV3YS/decimals-gt-zero");
```
https://github.com/pooltogether/aave-v3-yield-source/blob/e63d1b0e396a5bce89f093630c282ca1c6627e44/contracts/AaveV3YieldSource.sol#L179

```solidity
File: contracts/AaveV3YieldSource.sol   #6

233       require(_shares > 0, "AaveV3YS/shares-gt-zero");
```
https://github.com/pooltogether/aave-v3-yield-source/blob/e63d1b0e396a5bce89f093630c282ca1c6627e44/contracts/AaveV3YieldSource.sol#L233

```solidity
File: contracts/AaveV3YieldSource.sol   #7

276       require(_to != address(0), "AaveV3YS/payee-not-zero-address");
```
https://github.com/pooltogether/aave-v3-yield-source/blob/e63d1b0e396a5bce89f093630c282ca1c6627e44/contracts/AaveV3YieldSource.sol#L276

```solidity
File: contracts/AaveV3YieldSource.sol   #8

337       require(_token != address(aToken), "AaveV3YS/forbid-aToken-transfer");
```
https://github.com/pooltogether/aave-v3-yield-source/blob/e63d1b0e396a5bce89f093630c282ca1c6627e44/contracts/AaveV3YieldSource.sol#L337

```solidity
File: contracts/AaveV3YieldSource.sol   #9

349       require(_token != address(aToken), "AaveV3YS/forbid-aToken-allowance");
```
https://github.com/pooltogether/aave-v3-yield-source/blob/e63d1b0e396a5bce89f093630c282ca1c6627e44/contracts/AaveV3YieldSource.sol#L349

```diff
diff --git a/AaveV3YieldSource.sol.orig b/AaveV3YieldSource.sol.new
index 3975311..0a01635 100644
--- a/AaveV3YieldSource.sol.orig
+++ b/AaveV3YieldSource.sol.new
@@ -26,6 +26,17 @@ contract AaveV3YieldSource is ERC20, IYieldSource, Manageable, ReentrancyGuard {
   using SafeMath for uint256;
   using SafeERC20 for IERC20;
 
+  error ATokenNotZeroAddress();
+  error RCNotZeroAddress();
+  error PRNotZeroAddress();
+  error OwnerNotZeroAddress();
+  error DecimalsGtZero();
+  error SharesGtZero();
+  error PayeeNotZeroAddress();
+  error ForbidATokenTransfer();
+  error ForbidATokenAllowance();
+
+
   /* ============ Events ============ */
 
   /**
@@ -165,18 +176,18 @@ contract AaveV3YieldSource is ERC20, IYieldSource, Manageable, ReentrancyGuard {
     uint8 decimals_,
     address _owner
   ) Ownable(_owner) ERC20(_name, _symbol) ReentrancyGuard() {
-    require(address(_aToken) != address(0), "AaveV3YS/aToken-not-zero-address");
+    if (address(_aToken) == address(0)) revert ATokenNotZeroAddress();
     aToken = _aToken;
 
-    require(address(_rewardsController) != address(0), "AaveV3YS/RC-not-zero-address");
+    if (address(_rewardsController) == address(0)) revert RCNotZeroAddress();
     rewardsController = _rewardsController;
 
-    require(address(_poolAddressesProviderRegistry) != address(0), "AaveV3YS/PR-not-zero-address");
+    if (address(_poolAddressesProviderRegistry) == address(0)) revert PRNotZeroAddress();
     poolAddressesProviderRegistry = _poolAddressesProviderRegistry;
 
-    require(_owner != address(0), "AaveV3YS/owner-not-zero-address");
+    if (_owner == address(0)) revert OwnerNotZeroAddress();
 
-    require(decimals_ > 0, "AaveV3YS/decimals-gt-zero");
+    if (decimals_ == 0) revert DecimalsGtZero();
     _decimals = decimals_;
 
     // Approve once for max amount
@@ -230,7 +241,7 @@ contract AaveV3YieldSource is ERC20, IYieldSource, Manageable, ReentrancyGuard {
    */
   function supplyTokenTo(uint256 _depositAmount, address _to) external override nonReentrant {
     uint256 _shares = _tokenToShares(_depositAmount);
-    require(_shares > 0, "AaveV3YS/shares-gt-zero");
+    if (_shares == 0) revert SharesGtZero();
 
     address _underlyingAssetAddress = _tokenAddress();
     IERC20(_underlyingAssetAddress).safeTransferFrom(msg.sender, address(this), _depositAmount);
@@ -273,7 +284,7 @@ contract AaveV3YieldSource is ERC20, IYieldSource, Manageable, ReentrancyGuard {
    * @return True if operation was successful.
    */
   function claimRewards(address _to) external onlyManagerOrOwner returns (bool) {
-    require(_to != address(0), "AaveV3YS/payee-not-zero-address");
+    if (_to == address(0)) revert PayeeNotZeroAddress();
 
     address[] memory _assets = new address[](1);
     _assets[0] = address(aToken);
@@ -334,7 +345,7 @@ contract AaveV3YieldSource is ERC20, IYieldSource, Manageable, ReentrancyGuard {
     address _to,
     uint256 _amount
   ) external onlyManagerOrOwner {
-    require(address(_token) != address(aToken), "AaveV3YS/forbid-aToken-transfer");
+    if (address(_token) == address(aToken)) revert ForbidATokenTransfer();
     _token.safeTransfer(_to, _amount);
     emit TransferredERC20(msg.sender, _to, _amount, _token);
   }
@@ -346,7 +357,7 @@ contract AaveV3YieldSource is ERC20, IYieldSource, Manageable, ReentrancyGuard {
    * @param _token Address of the ERC20 token to check
    */
   function _requireNotAToken(address _token) internal view {
-    require(_token != address(aToken), "AaveV3YS/forbid-aToken-allowance");
+    if (_token == address(aToken)) revert ForbidATokenTransfer();
   }
 
   /**
```

```diff
diff --git a/gas.orig b/gas.new
index d87edc2..3aa8ff3 100644
--- a/gas.orig
+++ b/gas.new
@@ -17,7 +17,7 @@
 ·····························|··························|·············|·············|·············|···············|··············
 |  AaveV3YieldSourceHarness  ·  setManager              ·          -  ·          -  ·      47982  ·            4  ·          -  │
 ·····························|··························|·············|·············|·············|···············|··············
-|  AaveV3YieldSourceHarness  ·  supplyTokenTo           ·     110668  ·     153720  ·     142217  ·            6  ·          -  │
+|  AaveV3YieldSourceHarness  ·  supplyTokenTo           ·     110662  ·     153714  ·     142211  ·            6  ·          -  │
 ·····························|··························|·············|·············|·············|···············|··············
 |  AaveV3YieldSourceHarness  ·  transferERC20           ·      58318  ·      60467  ·      59393  ·            2  ·          -  │
 ·····························|··························|·············|·············|·············|···············|··············
@@ -29,7 +29,7 @@
 ·····························|··························|·············|·············|·············|···············|··············
 |  Deployments                                          ·                                         ·  % of limit   ·             │
 ························································|·············|·············|·············|···············|··············
-|  AaveV3YieldSourceHarness                             ·    2505760  ·    2505772  ·    2505770  ·        8.4 %  ·          -  │
+|  AaveV3YieldSourceHarness                             ·    2461083  ·    2461095  ·    2461093  ·        8.2 %  ·          -  │
 ························································|·············|·············|·············|···············|··············
 |  ERC20Mintable                                        ·          -  ·          -  ·    1154099  ·        3.8 %  ·          -  │
 ·-------------------------------------------------------|-------------|-------------|-------------|---------------|-------------·
```

### 8. Functions guaranteed to revert when called by normal users can be marked `payable`
If a function modifier such as `onlyOwner` is used, the function will revert if a normal user tries to pay the function. Marking the function as `payable` will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided. The extra opcodes avoided are 
`CALLVALUE`(2),`DUP1`(3),`ISZERO`(3),`PUSH2`(3),`JUMPI`(10),`PUSH1`(3),`DUP1`(3),`REVERT`(0),`JUMPDEST`(1),`POP`(2), which costs an average of about 21 gas per call to the function, in addition to the extra deployment cost

```solidity
File: contracts/AaveV3YieldSource.sol   #1

275     function claimRewards(address _to) external onlyManagerOrOwner returns (bool) {
```
https://github.com/pooltogether/aave-v3-yield-source/blob/e63d1b0e396a5bce89f093630c282ca1c6627e44/contracts/AaveV3YieldSource.sol#L275

```solidity
File: contracts/AaveV3YieldSource.sol   #2

296     function decreaseERC20Allowance(
297       IERC20 _token,
298       address _spender,
299       uint256 _amount
300     ) external onlyManagerOrOwner {
```
https://github.com/pooltogether/aave-v3-yield-source/blob/e63d1b0e396a5bce89f093630c282ca1c6627e44/contracts/AaveV3YieldSource.sol#L296-L300

```solidity
File: contracts/AaveV3YieldSource.sol   #3

315     function increaseERC20Allowance(
316       IERC20 _token,
317       address _spender,
318       uint256 _amount
319     ) external onlyManagerOrOwner {
```
https://github.com/pooltogether/aave-v3-yield-source/blob/e63d1b0e396a5bce89f093630c282ca1c6627e44/contracts/AaveV3YieldSource.sol#L315-L319

```solidity
File: contracts/AaveV3YieldSource.sol   #4

332     function transferERC20(
333       IERC20 _token,
334       address _to,
335       uint256 _amount
336     ) external onlyManagerOrOwner {
```
https://github.com/pooltogether/aave-v3-yield-source/blob/e63d1b0e396a5bce89f093630c282ca1c6627e44/contracts/AaveV3YieldSource.sol#L332-L336

```diff
diff --git a/AaveV3YieldSource.sol.orig b/AaveV3YieldSource.sol.new
index 3975311..b16b82a 100644
--- a/AaveV3YieldSource.sol.orig
+++ b/AaveV3YieldSource.sol.new
@@ -272,7 +272,7 @@ contract AaveV3YieldSource is ERC20, IYieldSource, Manageable, ReentrancyGuard {
    * @param _to Address where the claimed rewards will be sent
    * @return True if operation was successful.
    */
-  function claimRewards(address _to) external onlyManagerOrOwner returns (bool) {
+  function claimRewards(address _to) payable external onlyManagerOrOwner returns (bool) {
     require(_to != address(0), "AaveV3YS/payee-not-zero-address");
 
     address[] memory _assets = new address[](1);
@@ -297,7 +297,7 @@ contract AaveV3YieldSource is ERC20, IYieldSource, Manageable, ReentrancyGuard {
     IERC20 _token,
     address _spender,
     uint256 _amount
-  ) external onlyManagerOrOwner {
+  ) payable external onlyManagerOrOwner {
     _requireNotAToken(address(_token));
     _token.safeDecreaseAllowance(_spender, _amount);
     emit DecreasedERC20Allowance(msg.sender, _spender, _amount, _token);
@@ -316,7 +316,7 @@ contract AaveV3YieldSource is ERC20, IYieldSource, Manageable, ReentrancyGuard {
     IERC20 _token,
     address _spender,
     uint256 _amount
-  ) external onlyManagerOrOwner {
+  ) payable external onlyManagerOrOwner {
     _requireNotAToken(address(_token));
     _token.safeIncreaseAllowance(_spender, _amount);
     emit IncreasedERC20Allowance(msg.sender, _spender, _amount, _token);
@@ -333,7 +333,7 @@ contract AaveV3YieldSource is ERC20, IYieldSource, Manageable, ReentrancyGuard {
     IERC20 _token,
     address _to,
     uint256 _amount
-  ) external onlyManagerOrOwner {
+  ) payable external onlyManagerOrOwner {
     require(address(_token) != address(aToken), "AaveV3YS/forbid-aToken-transfer");
     _token.safeTransfer(_to, _amount);
     emit TransferredERC20(msg.sender, _to, _amount, _token);
```

```diff
diff --git a/gas.orig b/gas.new
index d87edc2..b10b8b3 100644
--- a/gas.orig
+++ b/gas.new
@@ -5,11 +5,11 @@
 ·····························|··························|·············|·············|·············|···············|··············
 |  Contract                  ·  Method                  ·  Min        ·  Max        ·  Avg        ·  # calls      ·  usd (avg)  │
 ·····························|··························|·············|·············|·············|···············|··············
-|  AaveV3YieldSourceHarness  ·  claimRewards            ·      54735  ·      56896  ·      55816  ·            4  ·          -  │
+|  AaveV3YieldSourceHarness  ·  claimRewards            ·      54711  ·      56872  ·      55792  ·            4  ·          -  │
 ·····························|··························|·············|·············|·············|···············|··············
-|  AaveV3YieldSourceHarness  ·  decreaseERC20Allowance  ·      37465  ·      39638  ·      38910  ·            3  ·          -  │
+|  AaveV3YieldSourceHarness  ·  decreaseERC20Allowance  ·      37441  ·      39614  ·      38886  ·            3  ·          -  │
 ·····························|··························|·············|·············|·············|···············|··············
-|  AaveV3YieldSourceHarness  ·  increaseERC20Allowance  ·      59133  ·      61618  ·      61212  ·            7  ·          -  │
+|  AaveV3YieldSourceHarness  ·  increaseERC20Allowance  ·      59109  ·      61594  ·      61188  ·            7  ·          -  │
 ·····························|··························|·············|·············|·············|···············|··············
 |  AaveV3YieldSourceHarness  ·  mint                    ·      51381  ·      68493  ·      61904  ·           13  ·          -  │
 ·····························|··························|·············|·············|·············|···············|··············
@@ -19,7 +19,7 @@
 ·····························|··························|·············|·············|·············|···············|··············
 |  AaveV3YieldSourceHarness  ·  supplyTokenTo           ·     110668  ·     153720  ·     142217  ·            6  ·          -  │
 ·····························|··························|·············|·············|·············|···············|··············
-|  AaveV3YieldSourceHarness  ·  transferERC20           ·      58318  ·      60467  ·      59393  ·            2  ·          -  │
+|  AaveV3YieldSourceHarness  ·  transferERC20           ·      58294  ·      60443  ·      59369  ·            2  ·          -  │
 ·····························|··························|·············|·············|·············|···············|··············
 |  ERC20Mintable             ·  approve                 ·      26692  ·      46592  ·      43275  ·            6  ·          -  │
 ·····························|··························|·············|·············|·············|···············|··············
@@ -29,7 +29,7 @@
 ·····························|··························|·············|·············|·············|···············|··············
 |  Deployments                                          ·                                         ·  % of limit   ·             │
 ························································|·············|·············|·············|···············|··············
-|  AaveV3YieldSourceHarness                             ·    2505760  ·    2505772  ·    2505770  ·        8.4 %  ·          -  │
+|  AaveV3YieldSourceHarness                             ·    2586991  ·    2587003  ·    2587001  ·        8.6 %  ·          -  │
 ························································|·············|·············|·············|···············|··············
 |  ERC20Mintable                                        ·          -  ·          -  ·    1154099  ·        3.8 %  ·          -  │
 ·-------------------------------------------------------|-------------|-------------|-------------|---------------|-------------·
```

### 9. Method IDs can be fiddled with to reduce gas costs
See [this](https://medium.com/joyso/solidity-how-does-function-name-affect-gas-consumption-in-smart-contract-47d270d8ac92) page for details

```solidity
File: contracts/AaveV3YieldSource.sol (various lines)   #1

```
https://github.com/pooltogether/aave-v3-yield-source/blob/e63d1b0e396a5bce89f093630c282ca1c6627e44/contracts/AaveV3YieldSource.sol

```diff
diff --git a/AaveV3YieldSource.sol.orig b/AaveV3YieldSource.sol.new
index 3975311..dc04818 100644
--- a/AaveV3YieldSource.sol.orig
+++ b/AaveV3YieldSource.sol.new
@@ -180,7 +180,7 @@ contract AaveV3YieldSource is ERC20, IYieldSource, Manageable, ReentrancyGuard {
     _decimals = decimals_;
 
     // Approve once for max amount
-    IERC20(_tokenAddress()).safeApprove(address(_pool()), type(uint256).max);
+    IERC20(_tokenAddress()).safeApprove(address(_pool_Jo$()), type(uint256).max);
 
     emit AaveV3YieldSourceInitialized(
       _aToken,
@@ -234,7 +234,7 @@ contract AaveV3YieldSource is ERC20, IYieldSource, Manageable, ReentrancyGuard {
 
     address _underlyingAssetAddress = _tokenAddress();
     IERC20(_underlyingAssetAddress).safeTransferFrom(msg.sender, address(this), _depositAmount);
-    _pool().supply(_underlyingAssetAddress, _depositAmount, address(this), REFERRAL_CODE);
+    _pool_Jo$().supply(_underlyingAssetAddress, _depositAmount, address(this), REFERRAL_CODE);
 
     _mint(_to, _shares);
 
@@ -256,7 +256,7 @@ contract AaveV3YieldSource is ERC20, IYieldSource, Manageable, ReentrancyGuard {
     _burn(msg.sender, _shares);
 
     uint256 _beforeBalance = _assetToken.balanceOf(address(this));
-    _pool().withdraw(_underlyingAssetAddress, _redeemAmount, address(this));
+    _pool_Jo$().withdraw(_underlyingAssetAddress, _redeemAmount, address(this));
     uint256 _afterBalance = _assetToken.balanceOf(address(this));
 
     uint256 _balanceDiff = _afterBalance.sub(_beforeBalance);
@@ -385,7 +385,7 @@ contract AaveV3YieldSource is ERC20, IYieldSource, Manageable, ReentrancyGuard {
    * @notice Retrieves Aave PoolAddressesProvider address.
    * @return A reference to PoolAddressesProvider interface.
    */
-  function _poolProvider() internal view returns (IPoolAddressesProvider) {
+  function _poolProvider_uaF() internal view returns (IPoolAddressesProvider) {
     return
       IPoolAddressesProvider(
         poolAddressesProviderRegistry.getAddressesProvidersList()[ADDRESSES_PROVIDER_ID]
@@ -396,7 +396,7 @@ contract AaveV3YieldSource is ERC20, IYieldSource, Manageable, ReentrancyGuard {
    * @notice Retrieves Aave Pool address.
    * @return A reference to Pool interface.
    */
-  function _pool() internal view returns (IPool) {
-    return IPool(_poolProvider().getPool());
+  function _pool_Jo$() internal view returns (IPool) {
+    return IPool(_poolProvider_uaF().getPool());
   }
 }
```

```diff
diff --git a/gas.orig b/gas.new
index d87edc2..72bf242 100644
--- a/gas.orig
+++ b/gas.new
@@ -29,7 +29,7 @@
 ·····························|··························|·············|·············|·············|···············|··············
 |  Deployments                                          ·                                         ·  % of limit   ·             │
 ························································|·············|·············|·············|···············|··············
-|  AaveV3YieldSourceHarness                             ·    2505760  ·    2505772  ·    2505770  ·        8.4 %  ·          -  │
+|  AaveV3YieldSourceHarness                             ·    2505748  ·    2505760  ·    2505758  ·        8.4 %  ·          -  │
 ························································|·············|·············|·············|···············|··············
 |  ERC20Mintable                                        ·          -  ·          -  ·    1154099  ·        3.8 %  ·          -  │
 ·-------------------------------------------------------|-------------|-------------|-------------|---------------|-------------·
```
