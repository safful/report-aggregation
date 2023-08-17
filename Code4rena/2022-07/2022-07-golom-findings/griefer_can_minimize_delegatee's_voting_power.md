## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed
- selected-for-report

# [Griefer can minimize delegatee's voting power](https://github.com/code-423n4/2022-07-golom-findings/issues/707) 

# Lines of code

https://github.com/code-423n4/2022-07-golom/blob/e5efa8f9d6dda92a90b8b2c4902320acf0c26816/contracts/vote-escrow/VoteEscrowDelegation.sol#L99
https://github.com/code-423n4/2022-07-golom/blob/e5efa8f9d6dda92a90b8b2c4902320acf0c26816/contracts/vote-escrow/VoteEscrowDelegation.sol#L71-L89


# Vulnerability details

## Impact
Similar to a previous submission, there are no checks preventing against delegating the same lock NFT multiple times. This opens an avenue to an expensive but potentially profitable griefing attack where the malicious user fills the victim's delegated token array with minimum voting power. The attacker can ensure that a delegatee has 0 voting power.

## Proof of Concept
Taking a look at the `delegate()` function below, there are no checks that a lock NFT has not already been delegated. Therefore, an attacker can delegate their token with minimum voting power (threshold initialized with value 0) to the victim. 

```
    function delegate(uint256 tokenId, uint256 toTokenId) external {
        require(ownerOf(tokenId) == msg.sender, 'VEDelegation: Not allowed');
        require(this.balanceOfNFT(tokenId) >= MIN_VOTING_POWER_REQUIRED, 'VEDelegation: Need more voting power');


        delegates[tokenId] = toTokenId;
        uint256 nCheckpoints = numCheckpoints[toTokenId];


        if (nCheckpoints > 0) {
            Checkpoint storage checkpoint = checkpoints[toTokenId][nCheckpoints - 1];
            checkpoint.delegatedTokenIds.push(tokenId);
            _writeCheckpoint(toTokenId, nCheckpoints, checkpoint.delegatedTokenIds);
        } else {
            uint256[] memory array = new uint256[](1);
            array[0] = tokenId;
            _writeCheckpoint(toTokenId, nCheckpoints, array);
        }


        emit DelegateChanged(tokenId, toTokenId, msg.sender);
    }
```

There is a limit of 500 delegated tokens per delegatee. Therefore, the attacker can ensure minimum voting power if they delegate a worthless token 500 times to the victim:

```
    function _writeCheckpoint(
        uint256 toTokenId,
        uint256 nCheckpoints,
        uint256[] memory _delegatedTokenIds
    ) internal {
        require(_delegatedTokenIds.length < 500, 'VVDelegation: Cannot stake more');
```

A more likely scenario would be as follows:
- A proposal is live.
- Users delegate their voting power to addresses of their choosing.
- A and B are around the same voting power.
- A and B both have 400 delegatees.
- Malicious address A delegates minimum voting power 100 times to fill B's array to 500.
- Address A can self-delegate just a bit more to obtain more voting power.


## Tools Used
Manual review.

## Recommended Mitigation Steps
Firstly, removing the ability to delegate the same lock NFT would make this griefing attack much more expensive. Even if that is patched, a griefing attack is still possible by simply creating more locks and delegating them all once. 

I believe that removing the 500 delegated token limit would prove to mitigate this issue.