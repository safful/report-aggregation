## Tags

- bug
- duplicate
- disagree with severity
- QA (Quality Assurance)
- sponsor confirmed

# [In RariMerkleRedeemer, function signAndClaim() doesn't have hasNotSigned and has different behavior than sign() and signAndClaimAndRedeem() ](https://github.com/code-423n4/2022-09-tribe-findings/issues/107) 

# Lines of code

https://github.com/code-423n4/2022-09-tribe/blob/769b0586b4975270b669d7d1581aa5672d6999d5/contracts/shutdown/fuse/RariMerkleRedeemer.sol#L88-L98
https://github.com/code-423n4/2022-09-tribe/blob/769b0586b4975270b669d7d1581aa5672d6999d5/contracts/shutdown/fuse/RariMerkleRedeemer.sol#L108-L118
https://github.com/code-423n4/2022-09-tribe/blob/769b0586b4975270b669d7d1581aa5672d6999d5/contracts/shutdown/fuse/RariMerkleRedeemer.sol#L48-L50


# Vulnerability details

## Impact
All three functions `signAndClaim()`, `sign()` and `signAndClaimAndRedeem()` are signing but `signAndClaim()` has different modifier than the other two. function `signAndClaim()` doesn't have `hasNotSigned` modifier and it's callable even when the users already signed. this different access level and behavior can cause other security issues. for example here it's possible for user to run sign multiple times.

## Proof of Concept
This is `signAndClaim()`, `sign()` and `signAndClaimAndRedeem()` codes in `RariMerkleRedeemer`:
```
    function sign(bytes calldata signature) external override hasNotSigned nonReentrant {
        _sign(signature);
    }

    function signAndClaim(
        bytes calldata signature,
        address[] calldata cTokens,
        uint256[] calldata amounts,
        bytes32[][] calldata merkleProofs
    ) external override nonReentrant {
        // both sign and claim/multiclaim will revert on invalid signatures/proofs
        _sign(signature);
        _multiClaim(cTokens, amounts, merkleProofs);
    }

    function signAndClaimAndRedeem(
        bytes calldata signature,
        address[] calldata cTokens,
        uint256[] calldata amountsToClaim,
        uint256[] calldata amountsToRedeem,
        bytes32[][] calldata merkleProofs
    ) external override hasNotSigned nonReentrant {
        _sign(signature);
        _multiClaim(cTokens, amountsToClaim, merkleProofs);
        _multiRedeem(cTokens, amountsToRedeem);
    }
```
As you can see `signAndClaimAndRedeem()` and `sign()` has `hasNotSigned ` modifier but `signAndClaim` doesn't have that modifier.

## Tools Used
VIM

## Recommended Mitigation Steps
add same modifier for `signAndClaimAndRedeem()` too.