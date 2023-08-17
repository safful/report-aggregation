## Tags

- bug
- 3 (High Risk)
- sponsor confirmed
- selected-for-report

# [Old delegatee not deleted when delegating to new tokenId](https://github.com/code-423n4/2022-07-golom-findings/issues/169) 

# Lines of code

https://github.com/code-423n4/2022-07-golom/blob/8f198624b97addbbe9602a451c908ea51bd3357c/contracts/vote-escrow/VoteEscrowDelegation.sol#L80


# Vulnerability details

## Impact
In `delegate`, when a user delegates to a new tokenId, the tokenId is not removed from the current delegatee. Therefore, one user can easily multiply his voting power, which makes the toking useless for voting / governance decisions.

## Proof Of Concept
Bob owns the token with ID 1 with a current balance of 1000. He also owns tokens 2, 3, 4, 5. Therefore, he calls `delegate(1, 2)`, `delegate(1, 3)`, `delegate(1, 4)`, `delegate(1, 5)`. Now, if there is a governance decision and `getVotes` is called, Bobs balance of 1000 is included in token 2, 3, 4, and 5. Therefore, he quadrupled the voting power of token 1.

## Recommended Mitigation Steps
Remove the entry in `delegatedTokenIds` of the old delegatee or simply call `removeDelegation` first.

