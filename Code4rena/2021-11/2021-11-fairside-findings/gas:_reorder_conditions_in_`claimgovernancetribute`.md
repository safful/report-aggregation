## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Gas: Reorder conditions in `claimGovernanceTribute`](https://github.com/code-423n4/2021-11-fairside-findings/issues/76) 

# Handle

cmichel


# Vulnerability details

The `FSD.claimGovernanceTribute` function first performs the expensive `getPriorConvictionScore` instead of the cheap `isGovernance[msg.sender]` check.

```solidity
function claimGovernanceTribute(uint256 num) external {
    require(
        governanceThreshold <=
            getPriorConvictionScore(
                msg.sender,
                governanceTributes[num].blockNumber
            ) &&
            // @audit gas: rearrange this to be first for short circuiting
            isGovernance[msg.sender],
        "FSD::claimGovernanceTribute: Not a governance member"
    );
    _claimGovernanceTribute(num);
}
```

Reordering the conditions to first do the cheap governance check would allow this function to short-circuit if the user is not a governor, which will save gas on average.
The last assignment `membership[msg.sender] = user;` is not required.


