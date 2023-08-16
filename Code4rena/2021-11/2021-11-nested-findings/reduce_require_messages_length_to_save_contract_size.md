## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Reduce require messages length to save contract size](https://github.com/code-423n4/2021-11-nested-findings/issues/14) 

# Handle

0xngndev


# Vulnerability details

## Impact
Running a quick contract size check in the NestedFactory contract, I noticed it sat at 27590 bytes, exceeding the allowed 24576 bytes to deploy on mainnet. I removed the require messages alone in that contract and found the contract size dropped to 23172 bytes. Considering you are using large require messages in all the codebase, I would suggest considering a change of approach as to how you expose the error messages. I'll add my suggestions below.

## Recommended Mitigation Steps
Two ways:
1) Shorten the length of the string messages to just the error instead of including the contract and the function. UniswapV3 repo may be a good example of how to do this. You can always explain errors further in the natspec, or in your documentation (you can make a common errors section).
2) Change require statements for if (...) revert CustomError(). Per solidity docs:

"Using a custom error instance will usually be much cheaper than a string description, because you can use the name of the error to describe it, which is encoded in only four bytes. A longer description can be supplied via NatSpec which does not incur any costs."

Link: https://docs.soliditylang.org/en/v0.8.10/control-structures.html?highlight=error#revert

## Tools Used
dapptools make size

