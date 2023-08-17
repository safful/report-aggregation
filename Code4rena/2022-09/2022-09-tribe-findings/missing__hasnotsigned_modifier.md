## Tags

- bug
- duplicate
- disagree with severity
- QA (Quality Assurance)
- sponsor confirmed
- old-submission-method

# [missing  hasNotSigned modifier](https://github.com/code-423n4/2022-09-tribe-findings/issues/27) 

# Lines of code

https://github.com/code-423n4/2022-09-tribe/blob/769b0586b4975270b669d7d1581aa5672d6999d5/contracts/shutdown/fuse/RariMerkleRedeemer.sol#L88


# Vulnerability details

## Impact
signing more than once

## Proof of Concept
Other sign functions has hasSigned modifier

## Tools Used

## Recommended Mitigation Steps
function signAndClaim(
        bytes calldata signature,
        address[] calldata cTokens,
        uint256[] calldata amounts,
        bytes32[][] calldata merkleProofs
    ) external override **hasSigned** nonReentrant

