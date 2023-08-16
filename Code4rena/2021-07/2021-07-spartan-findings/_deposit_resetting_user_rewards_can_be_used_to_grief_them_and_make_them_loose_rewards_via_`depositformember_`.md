## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [_deposit resetting user rewards can be used to grief them and make them loose rewards via `depositForMember `](https://github.com/code-423n4/2021-07-spartan-findings/issues/66) 

# Handle

GalloDaSballo


# Vulnerability details

## Impact
The function `_deposit` sets `mapMemberSynth_lastTime` to a date in the future
https://github.com/code-423n4/2021-07-spartan/blob/e2555aab44d9760fdd640df9095b7235b70f035e/contracts/synthVault.sol#L107


`mapMemberSynth_lastTime` is also used to calculate rewards earned

`depositForMember` allows anyone, to "make a donation" for the member and cause that member to loose all their accrued rewards

This can't be used for personal gain, but can be used to bring misery to others.


## Proof of Concept
`depositForMember` https://github.com/code-423n4/2021-07-spartan/blob/e2555aab44d9760fdd640df9095b7235b70f035e/contracts/synthVault.sol#L95
and can be called by anyone

This will set the member  ```
 mapMemberSynth_lastTime[_member][_synth] = block.timestamp + minimumDepositTime; // Record deposit time (scope: member -> synth)
```

this can be continuously exploited to make members never earn any reward


## Recommended Mitigation Steps

This is the second submission under the same exploit
This can be mitigated by harvesting for the user right before changing `mapMemberSynth_lastTime[_member][_synth]`

https://github.com/code-423n4/2021-07-spartan/blob/e2555aab44d9760fdd640df9095b7235b70f035e/contracts/synthVault.sol#L107

