## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Reduce length of require error messages to save in deployment costs](https://github.com/code-423n4/2021-12-sublime-findings/issues/47) 

# Handle

0xngndev


# Vulnerability details

### Impact

Some of your contracts are quite large byte-wise and require the optimizer with low runs to not reach the code size limit of 24576 bytes. The code size of these contracts can be drastically reduced by shortening the length of your error messages, reducing their deployment cost.

The best example of contracts having unnecessary long erorr messages are `CreditLine.sol` and `PoolFactory.sol`. When it comes to factories, whose primary goal is to deploy contracts, reducing the cost  of doing so is something to keep in mind.

### Mitigation Steps

Reduce the length of your error strings. Error messages like:

`'PoolFactory::createPool - Repayment interval not within limits'`

Could be reduced to:

`Interval out of limits`

You could save even more by doing something similar to what Uniswap does. They have very short error messages like: `ST`, and they expand on what they mean in their documentation.

Another approach is to use revert with CustomErrors, something like this:

`if (....) revert CustomError()`

Following the example I used above, you could have a custom error message that says:

`OutOfBounds()` and expand what it means in the natspec.

Custom error messages are cheaper than strings error messages. Here's a snippet of Solidity's documentation about this:

> Using a **custom** **error** instance will usually be much cheaper than a string description, because you can use the name of the error to describe it, which is encoded in only four bytes. A longer description can be supplied via NatSpec which does not incur any costs.
>

