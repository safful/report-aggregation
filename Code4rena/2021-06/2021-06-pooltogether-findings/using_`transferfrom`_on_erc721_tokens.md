## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed
- disagree with severity

# [Using `transferFrom` on ERC721 tokens](https://github.com/code-423n4/2021-06-pooltogether-findings/issues/115) 

# Handle

shw


# Vulnerability details

## Impact

In the function `awardExternalERC721` of contract `PrizePool`, when awarding external ERC721 tokens to the winners, the `transferFrom` keyword is used instead of `safeTransferFrom`. If any winner is a contract and is not aware of incoming ERC721 tokens, the sent tokens could be locked.

## Proof of Concept

Referenced code:
[PrizePool.sol#L602](https://github.com/code-423n4/2021-06-pooltogether/blob/main/contracts/PrizePool.sol#L602)

## Recommended Mitigation Steps

Consider changing `transferFrom` to `safeTransferFrom` at line 602. However, it could introduce a DoS attack vector if any winner maliciously rejects the received ERC721 tokens to make the others unable to get their awards. Possible mitigations are to use a `try/catch` statement to handle error cases separately or provide a function for the pool owner to remove malicious winners manually if this happens.

