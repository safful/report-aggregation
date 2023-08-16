## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Using fixed length array as parameter type can avoid checks to save gas and improve consistency](https://github.com/code-423n4/2021-11-fairside-findings/issues/59) 

# Handle

WatchPug


# Vulnerability details

https://github.com/code-423n4/2021-11-fairside/blob/20c68793f48ee2678508b9d3a1bae917c007b712/contracts/network/FSDNetwork.sol#L482-L495

```solidity=482
function setAssessors(address[] calldata _assessors) external {
        require(
            msg.sender == GOVERNANCE_ADDRESS,
            "FSDNetwork::setAssessors: Insufficient Privileges"
        );

        uint256 assessorsLength = _assessors.length;
        require(
            assessorsLength == 3,
            "FSDNetwork::setAssessors: Number of assessors must be three"
        );

        assessors = _assessors;
    }
```

### Recommendation

Change to:

```solidity=482
function setAssessors(address[3] calldata _assessors) external {
        require(
            msg.sender == GOVERNANCE_ADDRESS,
            "FSDNetwork::setAssessors: Insufficient Privileges"
        );

        assessors = _assessors;
    }
```

