## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Use `else if` in for loops can save gas and simplify code](https://github.com/code-423n4/2021-11-fairside-findings/issues/60) 

# Handle

WatchPug


# Vulnerability details

The checks in the for loop can be changed to `else if` to save gas and make sure `msg.sender != sigAssessor`.

https://github.com/code-423n4/2021-11-fairside/blob/20c68793f48ee2678508b9d3a1bae917c007b712/contracts/network/FSDNetwork.sol#L616-L649

```solidity=616
function _isApprovedByAssessors(
        bytes memory sig,
        uint256 id,
        Action action
    ) private view returns (bool) {
        bytes32 digest = _hashTypedDataV4(
            keccak256(abi.encode(CSR_ACTION, id, action))
        );

        address sigAssessor = ECDSA.recover(digest, sig);
        uint256 assessorsLength = assessors.length;
        bool assessorOne;
        bool assessorTwo;

        for (uint256 i = 0; i < assessorsLength; i++) {
            if (msg.sender == assessors[i]) {
                assessorOne = true;
            }
            if (sigAssessor == assessors[i]) {
                assessorTwo = true;
            }
        }

        require(
            assessorOne && assessorTwo,
            "FSDNetwork::_isApprovedByAssessors: Not an Assessor"
        );
        require(
            msg.sender != sigAssessor,
            "FSDNetwork::_isApprovedByAssessors: Cannot be the single Assessor"
        );

        return true;
    }
```


### Recommendation

Change to:

```solidity=616
function _isApprovedByAssessors(
        bytes memory sig,
        uint256 id,
        Action action
    ) private view returns (bool) {
        bytes32 digest = _hashTypedDataV4(
            keccak256(abi.encode(CSR_ACTION, id, action))
        );

        address sigAssessor = ECDSA.recover(digest, sig);
        uint256 assessorsLength = assessors.length;
        bool assessorOne;
        bool assessorTwo;

        for (uint256 i = 0; i < assessorsLength; i++) {
            if (msg.sender == assessors[i]) {
                assessorOne = true;
            } else if (sigAssessor == assessors[i]) {
                assessorTwo = true;
            }
        }

        require(
            assessorOne && assessorTwo,
            "FSDNetwork::_isApprovedByAssessors: Not an Assessor"
        );

        return true;
    }
```

