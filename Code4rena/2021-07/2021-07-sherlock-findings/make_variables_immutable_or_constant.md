## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Make variables immutable or constant](https://github.com/code-423n4/2021-07-sherlock-findings/issues/1) 

# Handle

bw


# Vulnerability details

## Impact
The `AaveV2.sol` contract made use of a number of `public` variables that were set only in the constructor and would remain constant. These variables were consuming storage slots, which unnecessarily increased the deployment and runtime gas costs of the contract.

## Proof of Concept

### Code Diff

```
diff --git a/contracts/strategies/AaveV2.sol b/contracts/strategies/AaveV2.sol
index 1b6ed56..9592986 100644
--- a/contracts/strategies/AaveV2.sol
+++ b/contracts/strategies/AaveV2.sol
@@ -21,15 +21,15 @@ import '../interfaces/IStrategy.sol';
 contract AaveV2 is IStrategy, Ownable {
   using SafeMath for uint256;
 
-  ILendingPoolAddressesProvider public lpAddressProvider =
+  ILendingPoolAddressesProvider public constant lpAddressProvider =
     ILendingPoolAddressesProvider(0xB53C1a33016B2DC2fF3653530bfF1848a515c8c5);
-  IAaveIncentivesController public aaveIncentivesController;
+  IAaveIncentivesController public immutable aaveIncentivesController;
 
-  ERC20 public override want;
-  IAToken public aWant;
+  ERC20 public immutable override want;
+  IAToken public immutable aWant;
 
-  address public sherlock;
-  address public aaveLmReceiver;
+  address public immutable sherlock;
+  address public immutable aaveLmReceiver;
 
   modifier onlySherlock() {
     require(msg.sender == sherlock, 'sherlock');
@@ -49,7 +49,7 @@ contract AaveV2 is IStrategy, Ownable {
     aaveLmReceiver = _aaveLmReceiver;
 
     ILendingPool lp = getLp();
-    want.approve(address(lp), uint256(-1));
+    ERC20(_aWant.UNDERLYING_ASSET_ADDRESS()).approve(address(lp), uint256(-1));
   }
```

### Gas Reporter Diff

```
diff --git a/base.rst b/immutable.rst
index 36b3138..79703fc 100644
--- a/base.rst
+++ b/immutable.rst
@@ -1,199 +1,199 @@
 ·----------------------------------------------------|---------------------------|-------------|-----------------------------·
 |                Solc version: 0.7.6                 ·  Optimizer enabled: true  ·  Runs: 200  ·  Block limit: 12450000 gas  │
 ·····················································|···························|·············|······························
-|  Methods                                           ·              100 gwei/gas               ·       2058.77 usd/eth       │
+|  Methods                                           ·              100 gwei/gas               ·       2061.52 usd/eth       │
 ·················|···································|·············|·············|·············|···············|··············
 |  Contract      ·  Method                           ·  Min        ·  Max        ·  Avg        ·  # calls      ·  usd (avg)  │
 ·················|···································|·············|·············|·············|···············|··············
-|  AaveV2        ·  claimRewards                     ·     397877  ·     437371  ·     417624  ·            2  ·      85.98  │
+|  AaveV2        ·  claimRewards                     ·     391566  ·     431060  ·     411313  ·            2  ·      84.79  │
 ·················|···································|·············|·············|·············|···············|··············
-|  AaveV2        ·  deposit                          ·          -  ·          -  ·     278549  ·            4  ·      57.35  │
+|  AaveV2        ·  deposit                          ·          -  ·          -  ·     274205  ·            4  ·      56.53  │
 ·················|···································|·············|·············|·············|···············|··············
-|  AaveV2        ·  withdraw                         ·     260005  ·     294205  ·     277105  ·            2  ·      57.05  │
+|  AaveV2        ·  withdraw                         ·     253692  ·     287892  ·     270792  ·            2  ·      55.82  │
 ·················|···································|·············|·············|·············|···············|··············
-|  AaveV2        ·  withdrawAll                      ·      59690  ·     275814  ·     167752  ·            2  ·      34.54  │
+|  AaveV2        ·  withdrawAll                      ·      53349  ·     267370  ·     160360  ·            2  ·      33.06  │
 ·················|···································|·············|·············|·············|···············|··············
|  Deployments                                       ·                                         ·  % of limit   ·             │
 ·····················································|·············|·············|·············|···············|··············
-|  AaveV2                                            ·          -  ·          -  ·     802248  ·        6.4 %  ·     165.16  │
+|  AaveV2                                            ·          -  ·          -  ·     762851  ·        6.1 %  ·     157.26  │
```

### Average Improvements

| Function      | Base      | Immutable | Diff    |
|---------------|-----------|-----------|---------|
| claimRewards  | 417624    | 411313    | -1.53%  |
| deposit       | 278549    | 274205    | -1.58%  |
| withdraw      | 277105    | 270792    | -2.33%  |
| withdrawAll   | 167752    | 160360    | -4.61%  |
| deployment    | 802248    | 762851    | -5.16%  |

By removing the `public` keyword from all variables that are not required (which are only used in the unit tests), the deployment costs can be further reduced to `700206`, which is an 14.57% reduction in gas costs.

## Tools Used

https://www.npmjs.com/package/hardhat-gas-reporter

## Recommended Mitigation Steps

Add the `immutable` key word to all variables that are only set during the constructor. Add the `constant` modifier for `lpAddressProvider`.

