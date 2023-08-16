## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Simplify function roll()](https://github.com/code-423n4/2021-11-overlay-findings/issues/68) 

# Handle

gpersoon


# Vulnerability details

## Impact
The function roll() of OverlayV1Comptroller.sol can be simplified.
This saves some gas and also makes the function easier to read. 
See below at "Recommended Mitigation Steps"

## Proof of Concept
https://github.com/code-423n4/2021-11-overlay/blob/914bed22f190ebe7088194453bab08c424c3f70c/contracts/market/OverlayV1Comptroller.sol#L352-L385
```JS
function roll( Roller[60] storage rollers, Roller memory _roller, uint _lastMoment, uint _cycloid ) internal returns ( uint cycloid_) {
        if (_roller.time != _lastMoment) {
            _cycloid += 1;
            if (_cycloid < CHORD) {
                rollers[_cycloid] = _roller;
            } else {
                _cycloid = 0;
                rollers[_cycloid] = _roller;
            }
        } else {
            rollers[_cycloid] = _roller;
        }
        cycloid_ = _cycloid;
    }
```

## Tools Used

## Recommended Mitigation Steps
Change the function to:
```JS
    function roll (Roller[60] storage rollers,Roller memory _roller,uint _lastMoment,uint _cycloid) internal returns (uint cycloid_) {
        if (_roller.time != _lastMoment) 
             _cycloid = (_cycloid + 1) % CHORD;                              
        rollers[_cycloid] = _roller;
        cycloid_ = _cycloid;
    }
```



