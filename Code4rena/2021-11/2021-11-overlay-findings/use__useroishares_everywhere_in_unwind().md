## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Use _userOiShares everywhere in unwind()](https://github.com/code-423n4/2021-11-overlay-findings/issues/78) 

# Handle

gpersoon


# Vulnerability details

## Impact
In the function unwind() of OverlayV1OVLCollateral.sol a tmp variable _userOiShares is used the store the value of _shares.
However _shares is still uses multiple times in the function.

Using _userOiShares everywhere would save gas.

## Proof of Concept
https://github.com/code-423n4/2021-11-overlay/blob/914bed22f190ebe7088194453bab08c424c3f70c/contracts/collateral/OverlayV1OVLCollateral.sol#L273-L336

```JS
 function unwind (  uint256 _positionId,  uint256 _shares ) external {
        require( 0 < _shares && _shares <= balanceOf(msg.sender, _positionId), "OVLV1:!shares");    // uses _shares
        ...
        uint _userOiShares = _shares; // move to start of the function
        uint _userNotional = _shares * pos.notional(_oi, _oiShares, _priceFrame) / _totalPosShares;    // uses _shares
        uint _userDebt = _shares * pos.debt / _totalPosShares;                                                          // uses _shares
        uint _userCost = _shares * pos.cost / _totalPosShares;                                                           // uses _shares
        uint _userOi = _shares * pos.oi(_oi, _oiShares) / _totalPosShares;                                          // uses _shares
...  
      _burn(msg.sender, _positionId, _shares);   // uses _shares
```

## Tools Used

## Recommended Mitigation Steps
Move "uint _userOiShares = _shares;" to the start of function unwind()
Replace all other instances of "_shares" with "_userOiShares"
 

