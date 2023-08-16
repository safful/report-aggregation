## Tags

- bug
- 2 (Med Risk)
- disagree with severity
- sponsor confirmed

# [isUnderwater returns opposite boolean for short positions](https://github.com/code-423n4/2021-11-overlay-findings/issues/53) 

# Handle

harleythedog


# Vulnerability details

## Impact
The function isUnderwater should return true iff the position value is < 0. In the case of a short position, this is when oi * (2 - priceFrame) - debt < 0 (based on the logic given in the _value function). Rearranging this equation, a short position is underwater iff oi * 2 < oi * priceFrame + debt. However, in the function _isUnderwater in Position.sol, the left and right side of this equation is flipped, meaning that the function will return the opposite of what it should when called on short positions.

Fortunately, the V1 implementation of OverlayOVLCollateral does not directly use the isUnderwater function in major control flow changes. However, line 304 of OverlayV1OVLCollateral.sol is a comment that says:

// TODO: think through edge case of underwater position ... and fee adjustments ...

which hints that this function is going to be used to deal with underwater positions. As a result, this issue would have a huge impact if not properly dealt with.

## Proof of Concept
See code for _isUnderwater here: https://github.com/code-423n4/2021-11-overlay/blob/1833b792caf3eb8756b1ba5f50f9c2ce085e54d0/contracts/libraries/Position.sol#L70

Notice that for short positions the inequality is flipped from what it should be (indeed, when self.debt is higher it is more likely that isUnder will be false, which is obviously incorrect).

Also, see the TODO comment here that shows isUndewater is important: https://github.com/code-423n4/2021-11-overlay/blob/1833b792caf3eb8756b1ba5f50f9c2ce085e54d0/contracts/collateral/OverlayV1OVLCollateral.sol#L304 

## Tools Used
Inspection

## Recommended Mitigation Steps
Flip the left and right side of the inequality for short positions in _isUnderwater.

