## Tags

- bug
- 2 (Med Risk)
- downgraded by judge
- primary issue
- satisfactory
- selected for report
- sponsor confirmed
- M-05

# [When  `rewardToken` is erc1155/erc777,an attacker can reenter and cause funds to be stuck in the contract forever](https://github.com/code-423n4/2023-01-rabbithole-findings/issues/523) 

# Lines of code

https://github.com/rabbitholegg/quest-protocol/blob/068d628f019e9469aecbf676370075c1f6c980fd/contracts/Quest.sol#L113-L116


# Vulnerability details

## Impact
If the reward token is `erc1155/erc777` an attacker can reenter and then buy/transfer another unclaimed token to the attacker address and then  the var 
`redeemTokens` wont be equal to how many tokens were actually redeemed. 

## Proof of Concept
ex:
reward token is an erc1155 that has  `_afterTokenTransfer` 
Alice(attacker) has 2  receipt tokens, the first one is on a  smart contract that will do the reentrancy, and the second  one is on Alice's address but is approved   to transfer to  the  smart contract(the own that holds the first receipt)
1. Alice calls the sc to `claim` rewards
```solidity
 IERC1155(rewardToken).safeTransferFrom(address(this), msg.sender, rewardAmountInWeiOrTokenId, amount_, '0x00');
``` 
2. `_afterTokenTransfer`  which causes the sc  to  call a function in its fallback function that transfers  the approved token to  the sc
```solidity
   try IERC1155Receiver(to).onERC1155Received(operator, from, id, amount, data) returns (bytes4 response) {
```
3. We then reenter with  recipient,not yet claimed token  and we claim it 
result:
 the invariant that `redeemedTokens` = tokens that are redeemed is false because it doesn't account for the first token that we reentered.
 The issue is worse  with `erc777` tokens because of  the fact  that accounting will be in  the `withdrawRemainingTokens` function
```solidity 

        uint256 unclaimedTokens = (receiptRedeemers() - redeemedTokens) * rewardAmountInWeiOrTokenId;
        uint256 nonClaimableTokens = IERC20(rewardToken).balanceOf(address(this)) - protocolFee() - unclaimedTokens;
        IERC20(rewardToken).safeTransfer(to_, nonClaimableTokens);

```
after the reentrancy 
ex: `redeemedTokens=9` but should be 10
`receiptRedeemers()=12` 
`rewardAmountInWeiOrTokenId=1e5`
`unclaimedTokens=300000 `
assuming they are some tokens left 
`balance(address(this)=201000` and `protocolFee=500`
`nonClaimableTokens=201000 - 500 - 300000` it would revert ( negative numbers  with uint) and   funds would be stuck in the contract forever
The real estimate for `nonClaimableTokens=201000-500-200000=500` and the owner can get funds out 
but 500 wei will be lost in the contract 
and  it can get worse with large amounts of quests and the attacker  reentering multiple times to cause a bigger gap between the real `redeemedTokens`  
## Tools Used

## Recommended Mitigation Steps
add  nonReentrancy modifier 