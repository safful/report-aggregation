## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Avoid unnecessary storage read can save gas](https://github.com/code-423n4/2021-10-covalent-findings/issues/49) 

# Handle

WatchPug


# Vulnerability details

https://github.com/code-423n4/2021-10-covalent/blob/ded3aeb2476da553e8bb1fe43358b73334434737/contracts/DelegatedStaking.sol#L101-L115

```solidity
function takeOutRewardTokens(uint128 amount) public onlyOwner {
    require(amount > 0, "Amount is 0");
    uint128 currentEpoch = uint128(block.number);
    uint128 epochs = amount / allocatedTokensPerEpoch;
    if (endEpoch != 0){
        require(endEpoch - epochs > currentEpoch, "Cannot takeout rewards from past");
        endEpoch = endEpoch - epochs;
    }
    else{
        require(rewardsLocked >= amount, "Amount is greater than available");
        rewardsLocked -= amount;
    }
    transferFromContract(owner(), amount);
    emit AllocatedTokensTaken(amount);
}
```

Since the `takeOutRewardTokens()` function is `onlyOwner`, `transferFromContract(owner(), amount);` can be changed to `transferFromContract(msg.sender, amount);` to avoid unnecessary internal call and storage read to save some gas.

