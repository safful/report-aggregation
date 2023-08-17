## Tags

- bug
- 2 (Med Risk)
- resolved
- sponsor confirmed

# [Staking `preSign` could use some basic validations](https://github.com/code-423n4/2022-06-yieldy-findings/issues/172) 

# Lines of code

https://github.com/code-423n4/2022-06-yieldy/blob/524f3b83522125fb7d4677fa7a7e5ba5a2c0fe67/src/contracts/Staking.sol#L769


# Vulnerability details

The function `preSign` acceps any `orderUid`
`function preSign(bytes calldata orderUid) external onlyOwner`

Because of how Cowswap works, accepting any `orderUid` can be used as a rug-vector.

This is because the orderData contains a `receiver` which in lack of validation could be any address.

You'd also be signing other parameters such as minOut and how long the order could be filled for, which you may or may not want to validate to give stronger security guarantees to end users.


## Recomended mitigation steps
I'd recommend adding basic validation for tokenOut, minOut and receiver.

Feel free to check the work we've done at Badger to validate order parameters, giving way stronger guarantees to end users.
https://github.com/GalloDaSballo/fair-selling/blob/44c0c0629289a0c4ccb3ca971cc5cd665ce5cb82/contracts/CowSwapSeller.sol#L194

Also notice how through the code above we are able to re-construct the `orderUid`, feel free to re-use that code which has been validated by the original Cowswap / GPv2 Developers

