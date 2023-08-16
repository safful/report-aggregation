## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Gas in `MathLib.sol:calculateQtyToReturnAfterFees()`: Avoid expensive calculation by checking if `_tokenASwapQty == 0 || _tokenBReserveQty == 0`](https://github.com/code-423n4/2022-01-elasticswap-findings/issues/48) 

# Handle

Dravee


# Vulnerability details

## Impact
Saving the gas cost from the calculation

## Proof of Concept
See the `@audit-info` tags:
```
File: MathLib.sol
141:     function calculateQtyToReturnAfterFees(
142:         uint256 _tokenASwapQty,
143:         uint256 _tokenAReserveQty,
144:         uint256 _tokenBReserveQty,
145:         uint256 _liquidityFeeInBasisPoints
146:     ) public pure returns (uint256 qtyToReturn) {
147:         uint256 tokenASwapQtyLessFee = //@audit-info == 0 if _tokenASwapQty == 0
148:             _tokenASwapQty * (BASIS_POINTS - _liquidityFeeInBasisPoints); 
149:         qtyToReturn =
150:             (tokenASwapQtyLessFee * _tokenBReserveQty) / //@audit-info 0 is possible if _tokenBReserveQty == 0 or above is equal to 0
151:             ((_tokenAReserveQty * BASIS_POINTS) + tokenASwapQtyLessFee);
152:     }
```

## Tools Used
VS Code

## Recommended Mitigation Steps
Return 0 if `_tokenASwapQty == 0 || _tokenBReserveQty == 0` or the `&&` equivalent.
Here's an example:
```
File: MathLib.sol
141:     function calculateQtyToReturnAfterFees(
142:         uint256 _tokenASwapQty,
143:         uint256 _tokenAReserveQty,
144:         uint256 _tokenBReserveQty,
145:         uint256 _liquidityFeeInBasisPoints
146:     ) public pure returns (uint256 qtyToReturn) {
147:         if(_tokenASwapQty != 0 && _tokenBReserveQty != 0){
148:             uint256 tokenASwapQtyLessFee = _tokenASwapQty * 
149:                 (BASIS_POINTS - _liquidityFeeInBasisPoints); 
150:             qtyToReturn = (tokenASwapQtyLessFee * _tokenBReserveQty) / 
151:                 ((_tokenAReserveQty * BASIS_POINTS) + tokenASwapQtyLessFee);
152:         }
153:     }
```
(here `qtyToReturn` if set to 0 by default so the value returned would be 0)

