## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [denial of service](https://github.com/code-423n4/2021-12-sublime-findings/issues/154) 

# Handle

certora


# Vulnerability details


https://github.com/code-423n4/2021-12-sublime/blob/main/contracts/Pool/Pool.sol#L645
if the borrow token is address(0) (ether), and someone calls withdrawLiquidity, it calls SavingsAccountUtil.transferTokens which will transfer to msg.sender, msg.value (of withdrawLiquidity, because it's an internal function). In other words, the liquidity provided will pay to themselves and their liquidity tokens will still be burned. therefore they will never be able to get their funds back.



## Recommended Mitigation Steps
the bug is in 
https://github.com/code-423n4/2021-12-sublime/blob/main/contracts/SavingsAccount/SavingsAccountUtil.sol
It is wrong to use msg.value in transferTokens because it'll be the msg.value of the calling function.
therefore every transfer of ether using this function is wrong and dangerous, the solution is to remove all msg.value from this function and just transfer _amount regularly.

