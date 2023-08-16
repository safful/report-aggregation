## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Use _brrrrdExpected everywhere in oiCap() ](https://github.com/code-423n4/2021-11-overlay-findings/issues/69) 

# Handle

gpersoon


# Vulnerability details

## Impact
The function oiCap() of OverlayV1Comptroller.sol save the value of brrrrdExpected in a tmp variable _brrrrdExpected.
Lateron brrrrdExpected is still used while _brrrrdExpected could also be used.
This saves a bit of gas.

## Proof of Concept
https://github.com/code-423n4/2021-11-overlay/blob/914bed22f190ebe7088194453bab08c424c3f70c/contracts/market/OverlayV1Comptroller.sol#L255-L279

```JS
 function oiCap() public virtual view returns (  uint cap_ ) {
    ...
        uint _brrrrdExpected = brrrrdExpected;
    ...
        cap_ = _surpassed ? 0 : _burnt || _expected
            ? _oiCap(false, depth(), staticCap, 0, 0)
            : _oiCap(true, depth(), staticCap, _brrrrd, brrrrdExpected);  // can also use _brrrrdExpected 
```
## Tools Used

## Recommended Mitigation Steps
Replace
```JS
     : _oiCap(true, depth(), staticCap, _brrrrd, brrrrdExpected);
```
with
```JS
     : _oiCap(true, depth(), staticCap, _brrrrd, _brrrrdExpected);
```

