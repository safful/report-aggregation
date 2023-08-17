## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [Truncation in `OrderValidator` can lead to resetting the fill and selling more tokens](https://github.com/code-423n4/2022-05-opensea-seaport-findings/issues/77) 

# Lines of code

https://github.com/code-423n4/2022-05-opensea-seaport/blob/4140473b1f85d0df602548ad260b1739ddd734a5/contracts/lib/OrderValidator.sol#L228
https://github.com/code-423n4/2022-05-opensea-seaport/blob/4140473b1f85d0df602548ad260b1739ddd734a5/contracts/lib/OrderValidator.sol#L231
https://github.com/code-423n4/2022-05-opensea-seaport/blob/4140473b1f85d0df602548ad260b1739ddd734a5/contracts/lib/OrderValidator.sol#L237
https://github.com/code-423n4/2022-05-opensea-seaport/blob/4140473b1f85d0df602548ad260b1739ddd734a5/contracts/lib/OrderValidator.sol#L238


# Vulnerability details

## Impact

A partial order's fractions (`numerator` and `denominator`) can be reset to `0` due to a truncation. This can be used to craft malicious orders:

1. Consider user Alice, who has 100 ERC1155 tokens, who approved all of their tokens to the `marketplaceContract`.
2. Alice places a `PARTIAL_OPEN` order with 10 ERC1155 tokens and consideration of ETH.
3. Malory tries to fill the order in the following way:
    1. Malory tries to fill 50% of the order, but instead of providing the fraction `1 / 2`, Bob provides `2**118 / 2**119`. This sets the `totalFilled` to `2**118` and `totalSize` to `2**119`.
    2. Malory tries to fill 10% of the order, by providing `1 / 10`. The computation `2**118 / 2**119 + 1 / 10` is done by "cross multiplying" the denominators, leading to the acutal fraction being `numerator = (2**118 * 10 + 2**119)` and `denominator = 2**119 * 10`.
    3. Because of the `uint120` truncation in [OrderValidator.sol#L228-L248](https://github.com/ProjectOpenSea/seaport/blob/6c24d09fc4be9bbecf749e6a7a592c8f7b659405/contracts/lib/OrderValidator.sol#L228-L248), the `numerator` and `denominator` are truncated to `0` and `0` respectively.
    4. Bob can now continue filling the order and draining any approved (1000 tokens in total) of the above ERC1155 tokens, for the same consideration amount! 

## Proof of Concept


For a full POC: https://gist.github.com/hrkrshnn/7c51b23f7c43c55ba0f8157c3b298409

The following change would make the above POC fail:
```diff
modified   contracts/lib/OrderValidator.sol
@@ -225,6 +225,8 @@ contract OrderValidator is Executor, ZoneInteraction {
                 // Update order status and fill amount, packing struct values.
                 _orderStatus[orderHash].isValidated = true;
                 _orderStatus[orderHash].isCancelled = false;
+                require(filledNumerator + numerator <= type(uint120).max, "overflow");
+                require(denominator <= type(uint120).max, "overflow");
                 _orderStatus[orderHash].numerator = uint120(
                     filledNumerator + numerator
                 );
@@ -234,6 +236,8 @@ contract OrderValidator is Executor, ZoneInteraction {
             // Update order status and fill amount, packing struct values.
             _orderStatus[orderHash].isValidated = true;
             _orderStatus[orderHash].isCancelled = false;
+            require(numerator <= type(uint120).max, "overflow");
+            require(denominator <= type(uint120).max, "overflow");
             _orderStatus[orderHash].numerator = uint120(numerator);
             _orderStatus[orderHash].denominator = uint120(denominator);
         }
```

## Tools Used

Manual review

## Recommended Mitigation Steps

A basic fix for this would involve adding the above checks for overflow / truncation and reverting in that case. However, we think the mechanism is still flawed in some respects and require more changes to fully fix it. See a related issue: "A malicious filler can fill a partial order in such a way that the rest cannot be filled by anyone" that points out a related but a more fundamental issue with the mechanism. 

