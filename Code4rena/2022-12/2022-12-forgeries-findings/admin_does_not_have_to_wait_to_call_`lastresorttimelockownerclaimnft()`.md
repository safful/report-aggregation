## Tags

- bug
- 3 (High Risk)
- primary issue
- satisfactory
- selected for report
- sponsor confirmed
- upgraded by judge
- H-01

# [Admin does not have to wait to call `lastResortTimelockOwnerClaimNFT()`](https://github.com/code-423n4/2022-12-forgeries-findings/issues/146) 

# Lines of code

https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L304-L320


# Vulnerability details

On contest page:
`"If no users ultimately claim the NFT, the admin specifies a timelock period after which they can retrieve the raffled NFT."`

Let's assume a recoverTimelock of 1 week.

The specification suggests that 1 week from the winner not having claimed the NFT. Meaning that the admin should only be able to call `lastResortTimelockOwnerClaimNFT()` only after `<block.timestamp at fulfillRandomWords()> + request.drawTimelock + 1 weeks`. 

Specification:
```
         drawTimelock                recoverTimelock
             │                              │
             ▼                              ▼
        ┌────┬──────────────────────────────┐
        │    │           1 week             │
        └────┴──────────────────────────────┘
        ▲
        │
fulfillRandomWords()
```
- The winner should have up to `drawTimelock` to claim before an admin can call `redraw()` and pick a new winner.
- The winner should have up to `recoverTimelock` to claim before an admin can call `lastResortTimelockOwnerClaimNFT()` to cancel the raffle.

But this is not the case. 

**recoverTimelock** is set in the `initialize(...)` function and not anywhere else. That means 1 week from initialization, the admin can call `lastResortTimelockOwnerClaimNFT()`. `redraw()` also does not update `recoverTimelock`.

In fact, `startDraw()` does not have to be called at the same time as `initialize(...)`. That means that if the draw was started after having been initialized for 1 week, the admin can withdraw at any time after that.

### Impact
Protocol does not work as intended.

### Recommendations
Just like for `drawTimelock`, `recoverTimelock` should also be updated for each dice roll.
`<block.timestamp at fulfillRandomWords()> + request.drawTimelock + <recoverBufferTime>`.  where `<recoverBufferTime>` is essentially the `drawBufferTime` currently used, but for `recoverTimelock`.

**Note:** currently, `drawTimelock` is updated in the `_requestRoll()` function. This is "technically less correct" as chainlink will take some time before `fulfillRandomWords(...)` callback. So the timelock is actually set before the winner has been chosen.  This should be insignificant under normal network conditions (Chainlink VRF shouldn't take > 1min) but both timelocks should be updated in the same function - either `_requestRoll()` or `fulfillRandomWords(...)`.