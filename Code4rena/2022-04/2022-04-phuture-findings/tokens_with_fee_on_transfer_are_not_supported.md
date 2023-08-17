## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Tokens with fee on transfer are not supported](https://github.com/code-423n4/2022-04-phuture-findings/issues/43) 

# Lines of code

https://github.com/code-423n4/2022-04-phuture/tree/main/contracts/IndexLogic.sol#L115


# Vulnerability details


There are ERC20 tokens that charge fee for every transfer() / transferFrom().

Vault.sol#addValue() assumes that the received amount is the same as the transfer amount, 
and uses it to calculate attributions, balance amounts, etc. 
But, the actual transferred amount can be lower for those tokens.
Therefore it's recommended to use the balance change before and after the transfer instead of the amount.
This way you also support the tokens with transfer fee - that are popular.


        https://github.com/code-423n4/2022-04-phuture/tree/main/contracts/IndexLogic.sol#L115

