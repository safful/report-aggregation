## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Global bounties variable and 0 bounty allow dos in bounty functionality of basket](https://github.com/code-423n4/2021-09-defiprotocol-findings/issues/25) 

# Handle

csanuragjain


# Vulnerability details

## Impact
It was observed that _bounties variable is global per basket. Also you are allowed to add 0 amount in bounty.
This means if user adds uint256 max times bounty with amount 0, no one can add further bounty on this basket

## Proof of Concept
1. User calls addBounty function with amount 0 uint256 max times

```
    function addBounty(IERC20 token, uint256 amount) public override returns (uint256) {
        // add bounty to basket
        token.safeTransferFrom(msg.sender, address(this), amount);
        _bounties.push(Bounty({
            token: address(token),
            amount: amount,
            active: true
        }));

        uint256 id = _bounties.length - 1;
        emit BountyAdded(token, amount, id);
        return id;
    }
```

2. Now noone can call bounty on this basket anymore

## Recommended Mitigation Steps
_bounties should be cleared once auction has been settled

