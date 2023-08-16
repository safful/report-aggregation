## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [`Auction.sol#settleAuction()` Mishandling bounty state could potentially disrupt `settleAuction()`](https://github.com/code-423n4/2021-09-defiprotocol-findings/issues/136) 

# Handle

WatchPug


# Vulnerability details

https://github.com/code-423n4/2021-09-defiProtocol/blob/main/contracts/contracts/Auction.sol#L143

```solidity=140
function withdrawBounty(uint256[] memory bountyIds) internal {
    // withdraw bounties
    for (uint256 i = 0; i < bountyIds.length; i++) {
        Bounty memory bounty = _bounties[bountyIds[i]];
        require(bounty.active);

        IERC20(bounty.token).transfer(msg.sender, bounty.amount);
        bounty.active = false;

        emit BountyClaimed(msg.sender, bounty.token, bounty.amount, bountyIds[i]);
    }
}
```

In the `withdrawBounty` function, `bounty.active` should be set to `false` when the bounty is claimed.

However, since `bounty` is stored in memory, the state update will not succeed.

### Impact

An auction successfully bonded by a regular user won't be able to be settled if they passed seemly active bountyIds, and the bonder will lose the bond.


### Proof of Concept

1. Create an auction;
2. Add a bounty;
3. Auction settled with bounty claimed;
4. Create a new auction;
5. Add a new bounty;
6. Calling `settleAuction()` with the bountyIds of the 2 seemly active bounties always reverts.

### Recommended Mitigation Steps

Change to:

```solidity=
Bounty storage bounty = _bounties[bountyIds[i]];
```

