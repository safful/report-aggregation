## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Anyone can unstake on behalf of someone](https://github.com/code-423n4/2021-07-sherlock-findings/issues/114) 

# Handle

cmichel


# Vulnerability details

The `PoolBase.unstakeWindowExpiry` function allows unstaking tokens of other users.
While the tokens are sent to the correct address, this can lead to issues with smart contracts that might rely on claiming the tokens themselves.

For example, suppose the `_to` address corresponds to a smart contract that has a function of the following form:

```solidity
function withdrawAndDoSomething() {
    uint256 amount = token.balanceOf(address(this));
    contract.unstakeWindowExpiry(address(this), id, token);
    amount = amount - token.balanceOf(address(this));
    token.transfer(externalWallet, amount)
}
```

If the contract has no other functions to transfer out funds, they may be locked forever in this contract.

