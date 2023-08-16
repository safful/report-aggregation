## Tags

- bug
- duplicate
- G (Gas Optimization)
- sponsor confirmed

# [Runtime constants not defined as immutable](https://github.com/code-423n4/2021-09-defiprotocol-findings/issues/15) 

# Handle

bw


# Vulnerability details

## Impact
The [`Factory.sol`](https://github.com/code-423n4/2021-09-defiProtocol/blob/main/contracts/contracts/Factory.sol#L19) contract made use of a number of `public` variables that were set only in the constructor and would remain constant. These variables were consuming storage slots, which unnecessarily increased the deployment and runtime gas costs of the contract.

For more information regarding the `immutable` keyword:
https://blog.soliditylang.org/2020/05/13/immutable-keyword/

## Proof of Concept

### Code Diff

```diff
diff --git a/contracts/contracts/Factory.sol b/contracts/contracts/Factory.sol
index 271945d..3bbdd4f 100644
--- a/contracts/contracts/Factory.sol
+++ b/contracts/contracts/Factory.sol
@@ -23,8 +23,8 @@ contract Factory is IFactory, Ownable {
 
     Proposal[] private _proposals;
 
-    IAuction public override auctionImpl;
-    IBasket public override basketImpl;
+    IAuction public immutable override auctionImpl;
+    IBasket public immutable override basketImpl;
 
     uint256 public override minLicenseFee = 1e15; // 1e15 0.1%
     uint256 public override auctionDecrement = 10000;
```

### Gas Improvement

```diff
diff --git a/base.gas b/factory-immutable.gas
index 9d48ade..1447433 100644
--- a/base.gas
+++ b/factory-immutable.gas
@@ -23,9 +23,9 @@
 ·····················|························|·············|·············|···········|···············|··············
 |  ERC20Upgradeable  ·  approve               ·          -  ·          -  ·    48900  ·            3  ·          -  │
 ·····················|························|·············|·············|···········|···············|··············
-|  Factory           ·  createBasket          ·     880031  ·     908831  ·   882911  ·           10  ·          -  │
+|  Factory           ·  createBasket          ·     875780  ·     904580  ·   878660  ·           10  ·          -  │
 ·····················|························|·············|·············|···········|···············|··············
-|  Factory           ·  proposeBasketLicense  ·     335488  ·     335512  ·   335505  ·           12  ·          -  │
+|  Factory           ·  proposeBasketLicense  ·     333388  ·     333412  ·   333405  ·           12  ·          -  │
 ·····················|························|·············|·············|···········|···············|··············
 |  Factory           ·  setOwnerSplit         ·          -  ·          -  ·    46173  ·            1  ·          -  │
 ·····················|························|·············|·············|···········|···············|··············
@@ -39,7 +39,7 @@
 ··············································|·············|·············|···········|···············|··············
 |  Basket                                     ·          -  ·          -  ·  2390793  ·          8 %  ·          -  │
 ··············································|·············|·············|···········|···············|··············
-|  Factory                                    ·          -  ·          -  ·  1706801  ·        5.7 %  ·          -  │
+|  Factory                                    ·          -  ·          -  ·  1684215  ·        5.6 %  ·          -  │
 ··············································|·············|·············|···········|···············|··············
 |  TestToken                                  ·     653145  ·     653193  ·   653163  ·        2.2 %  ·          -  │
 ·---------------------------------------------|-------------|-------------|-----------|---------------|-------------·
 ```

By removing the `public` keyword from all variables that are not required (which are only used in the unit tests), the deployment costs can be further reduced.

## Tools Used

https://www.npmjs.com/package/hardhat-gas-reporter

## Recommended Mitigation Steps

Add the immutable key word to all variables that are only set during the constructor. 

