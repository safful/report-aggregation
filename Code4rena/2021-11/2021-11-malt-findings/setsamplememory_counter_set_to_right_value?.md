## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [setSampleMemory counter set to right value?](https://github.com/code-423n4/2021-11-malt-findings/issues/193) 

# Handle

gpersoon


# Vulnerability details

## Impact
The function setSampleMemory of MovingAverage.sol takes the modulo of counter with the new value of _sampleMemory:
"counter = counter % _sampleMemory;"

Suppose: counter =15 ; sampleMemory=10 and  _sampleMemory=12
Then:   counter = counter % _sampleMemory ==> 3,  which means processing will continue at position 3.

However I think it should use: counter = counter % sampleMemory,  so it will continue at position 5

## Proof of Concept
https://github.com/code-423n4/2021-11-malt/blob/d3f6a57ba6694b47389b16d9d0a36a956c5e6a94/src/contracts/MovingAverage.sol#L424-L442

```JS
function setSampleMemory(uint256 _sampleMemory) external onlyRole(ADMIN_ROLE, "Must have admin privs")  {
  ...
    if (_sampleMemory > sampleMemory) {
      ...
      counter = counter % _sampleMemory;
    } else {
   }
    sampleMemory = _sampleMemory;
}
```

## Tools Used

## Recommended Mitigation Steps
Doublecheck the theory above and if you agree:
change
```JS
 counter = counter % _sampleMemory;
```
to
```JS
 counter = counter %  sampleMemory;
```


