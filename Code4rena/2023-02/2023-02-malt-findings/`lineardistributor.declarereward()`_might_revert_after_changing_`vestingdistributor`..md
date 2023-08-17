## Tags

- bug
- 2 (Med Risk)
- satisfactory
- selected for report
- sponsor confirmed
- M-08

# [`LinearDistributor.declareReward()` might revert after changing `vestingDistributor`.](https://github.com/code-423n4/2023-02-malt-findings/issues/28) 

# Lines of code

https://github.com/code-423n4/2023-02-malt/blob/main/contracts/RewardSystem/LinearDistributor.sol#L114
https://github.com/code-423n4/2023-02-malt/blob/main/contracts/RewardSystem/LinearDistributor.sol#L227


# Vulnerability details

## Impact
`LinearDistributor.declareReward()` might revert after changing `vestingDistributor` due to uint underflow.

## Proof of Concept
In `LinearDistributor.sol`, there is a [setVestingDistributor()](https://github.com/code-423n4/2023-02-malt/blob/main/contracts/RewardSystem/LinearDistributor.sol#L222-L228) function to update `vestingDistributor`.

And in `declareReward()`, it calculates the `netVest` and `netTime` by subtracting the previous amount and time.

```solidity
File: 2023-02-malt\contracts\RewardSystem\LinearDistributor.sol
112:     uint256 currentlyVested = vestingDistributor.getCurrentlyVested();
113: 
114:     uint256 netVest = currentlyVested - previouslyVested; //@audit revert after change vestingDistributor
115:     uint256 netTime = block.timestamp - previouslyVestedTimestamp;
116: 
```

But there is no guarantee that the vested amount of the new `vestingDistributor` is greater than the previously saved amount after changing the distributor.

Furthermore, there is no option to change `previouslyVested` beside this declareReward() function and it will keep reverting unless the admin change back the distributor.

## Tools Used
Manual Review

## Recommended Mitigation Steps
I think it would resolve the above problem if we change the previous amounts as well while updating the distributor.

```solidity
  function setVestingDistributor(address _vestingDistributor, uint _previouslyVested, uint _previouslyVestedTimestamp)
    external
    onlyRoleMalt(ADMIN_ROLE, "Must have admin privs")
  {
    require(_vestingDistributor != address(0), "SetVestDist: No addr(0)");
    vestingDistributor = IVestingDistributor(_vestingDistributor);

    previouslyVested = _previouslyVested;
    previouslyVestedTimestamp = _previouslyVestedTimestamp;
  }
```