## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Check for liquidation in value() ](https://github.com/code-423n4/2021-11-overlay-findings/issues/76) 

# Handle

gpersoon


# Vulnerability details

## Impact
The function value() of OverlayV1OVLCollateral.sol doesn't explicitly check for liquidated positions.
However because oiShares and debt are set to 0 during liquidation the resulting value will still be 0.

It seems more logical to check for liquidation in the beginning of the function and immediately return 0.
This saves gas for the situation where the function value() is called from another smart contract.

## Proof of Concept
 https://github.com/code-423n4/2021-11-overlay/blob/914bed22f190ebe7088194453bab08c424c3f70c/contracts/collateral/OverlayV1OVLCollateral.sol#L424-L448
```
function value (  uint _positionId ) public view returns ( uint256 value_) {
        Position.Info storage pos = positions[_positionId];
        IOverlayV1Market _market = IOverlayV1Market(pos.market);
        (   uint _oi,  uint _oiShares,   uint _priceFrame ) = _market.positionInfo( pos.isLong, pos.pricePoint );
        value_ = pos.value(  _oi, _oiShares,  _priceFrame );
}
```
 
## Tools Used

## Recommended Mitigation Steps
Add something like the following to function value():
```JS
        if (pos.oiShares == 0) return 0; // liquidated
```
 

