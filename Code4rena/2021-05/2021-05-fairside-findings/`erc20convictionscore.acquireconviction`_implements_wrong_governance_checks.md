## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed
- resolved

# [`ERC20ConvictionScore.acquireConviction` implements wrong governance checks](https://github.com/code-423n4/2021-05-fairside-findings/issues/45) 

# Handle

cmichel


# Vulnerability details

## Vulnerability Details

There are two issues with the governance checks when acquiring them from an NFT:

#### Missing balance check
The governance checks in `_updateConvictionScore` are:

```solidity
!isGovernance[user]
&& userConvictionScore >= governanceThreshold 
&& balanceOf(user) >= governanceMinimumBalance;
```

Whereas in `acquireConviction`, only `userConvictionScore >= governanceThreshold` is checked but not `&& balanceOf(user) >= governanceMinimumBalance`.

```solidity
else if (
    !isGovernance[msg.sender] && userNew >= governanceThreshold
) {
    isGovernance[msg.sender] = true;
}
```

#### the `wasGovernance` might be outdated

The second issue is that at the time of NFT creation, the `governanceThreshold` or `governanceMinimumBalance` was different and would not qualify for a governor now.
The NFT's governance state is blindly appplied to the new user:

```solidity
if (wasGovernance && !isGovernance[msg.sender]) {
    isGovernance[msg.sender] = true;
}
```

This allows a user to circumvent any governance parameter changes by front-running the change with an NFT creation.

## Impact
It's easy to circumvent the balance check to become a governor by minting and redeeming your own NFT.
One can also circumvent any governance parameter increases by front-running these actions with an NFT creation and backrunning with a redemption.

## Recommended Mitigation Steps
Add the missing balance check in `acquireConviction`.
Remove the `wasGovernance` governance transfer from the NFT and solely recompute it based on the current `governanceThreshold` / `governanceMinimumBalance` settings.


