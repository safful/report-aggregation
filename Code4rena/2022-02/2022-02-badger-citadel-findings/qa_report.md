## Tags

- bug
- QA (Quality Assurance)
- sponsor confirmed

# [QA Report](https://github.com/code-423n4/2022-02-badger-citadel-findings/issues/57) 

## Overall

We find the `2022-02-badger-citadel` codebase well-documented, well-structured, with a fair amount of tests, and pretty gas efficient.

There are 2 Non-critical issues and 1 Low severity issue found.

### [N1] Inconsistent style of error messages

Some of the error messages are prefixed with `TokenSale:` while some are not.

Consider updating the error messages to keep the style of error messages consistent.

### [N2] Tokens with fee on transfer are not supported

There are ERC20 tokens that charge fee for every `transfer()` or `transferFrom()`.

In the current implementation, `TokenSaleUpgradeable.sol#buy()` assumes that the received amount is the same as the transfer amount, and uses it to calculate `tokenOutAmount_`.

https://github.com/code-423n4/2022-02-badger-citadel/blob/main/contracts/TokenSaleUpgradeable.sol#L180-L183

Consider calling `balanceOf()` before and after the transfer to get the actual transferred amount if a token with transfer tax as `tokenIn` should be supported.

### [L3] Allowing `setTokenOutPrice` after the token sale starts can result in unexpected results for buyers

In the current implementation, the owner can call `setTokenOutPrice()` and change the tokenOutPrice anytime, including when the token sale already started. In the case of network congestion or in chance, if the owner `setTokenOutPrice()` to a higher price, it can result in unexpected tokenOutAmount for the buyers who submitted their `buy()` txs before but only get packed into the block after the `setTokenOutPrice()` tx.

We consider this an undesirable situation for users and there is pretty much no other way to prevent it, therefore it should be prevented at the smart contract level.

Consider making `setTokenOutPrice()` only callable before the sale starts.