## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [Funds cannot be withdrawn in `CoreCollection.withdraw`](https://github.com/code-423n4/2022-03-joyn-findings/issues/80) 

# Lines of code

https://github.com/code-423n4/2022-03-joyn/blob/main/core-contracts/contracts/CoreCollection.sol#L175


# Vulnerability details

 The `CoreCollection.withdraw` function uses `payableToken.transferFrom(address(this), msg.sender, amount)` to transfer tokens from the `CoreCollection` contract to the `msg.sender` ( who is the owner of the contract). The usage of `transferFrom` can result in serious issues. In fact, many ERC20 always require that in `transferFrom` `allowance[from][msg.sender] >= amount`, so in this case the call to the `withdraw` function will revert as the `allowance[CoreCollection][CoreCollection] == 0` and therefore the funds cannot ben withdrawn and will be locked forever in the contract.

Recommendation : replace `transferFrom` with `transfer`

