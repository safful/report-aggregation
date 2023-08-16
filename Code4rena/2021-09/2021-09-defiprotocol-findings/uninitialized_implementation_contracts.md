## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed

# [Uninitialized Implementation Contracts](https://github.com/code-423n4/2021-09-defiprotocol-findings/issues/18) 

# Handle

bw


# Vulnerability details

## Impact

The implementation contracts that are used by proxies are not initialized by default, this creates the possibility that the contracts will not be initialized after deployment.

Uninitialized implementations could result in Denial of Service exploits. This often involves initializing the contract so that it is possible to `delegatecall` into a contract that has the `selfdestruct` opcode. 

The contracts in-scope did not contain any `delegatecalls` that could be exploited. However, it is still regarded as best practice to ensure that the contracts cannot be initialized after deployment.

## Proof of Concept

As a defence in-depth measure, the implementations should be initialized during deployed by adding the following:
```diff
diff --git a/contracts/contracts/Auction.sol b/contracts/contracts/Auction.sol
index f07df8b..f7c21eb 100644
--- a/contracts/contracts/Auction.sol
+++ b/contracts/contracts/Auction.sol
@@ -44,6 +44,10 @@ contract Auction is IAuction {
         auctionOngoing = false;
     }
 
+    constructor() {
+        initialized = true;
+    }
+
     function initialize(address basket_, address factory_) public override {
         require(!initialized);
         basket = IBasket(basket_);
diff --git a/contracts/contracts/Basket.sol b/contracts/contracts/Basket.sol
index 5fef21b..4549365 100644
--- a/contracts/contracts/Basket.sol
+++ b/contracts/contracts/Basket.sol
@@ -33,6 +33,10 @@ contract Basket is IBasket, ERC20Upgradeable {
 
     uint256 public override lastFee;
 
+    constructor() {
+        __ERC20_init("",  "");
+    }
+
     function initialize(IFactory.Proposal memory proposal, IAuction auction_) public override {
         publisher = proposal.proposer;
         licenseFee = proposal.licenseFee;
```

* https://github.com/code-423n4/2021-09-defiProtocol/blob/main/contracts/contracts/Auction.sol#L9
* https://github.com/code-423n4/2021-09-defiProtocol/blob/main/contracts/contracts/Basket.sol#L12

## Tools Used

N/A

## Recommended Mitigation Steps

Initialize implementations during deployment by adding a constructor.




