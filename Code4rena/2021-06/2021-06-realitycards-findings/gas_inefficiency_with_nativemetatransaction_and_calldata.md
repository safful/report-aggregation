## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed
- resolved

# [Gas inefficiency with NativeMetaTransaction and calldata](https://github.com/code-423n4/2021-06-realitycards-findings/issues/1) 

# Handle

axic


# Vulnerability details

## Impact

In `lib/NativeMetaTransactions.sol` there is a frequently used helper `msgSender`:
```
    function msgSender() internal view returns (address payable sender) {
        if (msg.sender == address(this)) {
            bytes memory array = msg.data;
            uint256 index = msg.data.length;
            assembly {
                // Load the 32 bytes word from memory with the address on the lower 20 bytes, and mask those.
                sender := and(
                    mload(add(array, index)),
                    0xffffffffffffffffffffffffffffffffffffffff
                )
            }
        } else {
...
```

Even though only the last 20-bytes matter, the `bytes memory array = msg.data;` line causes the *entire* calldata to be copied to memory. This is exaggerated by the fact, that if `msgSender()` is called multiple times in a transaction, the calldata will be also copied multiple times as memory is not freed.

## Proof of Concept

N/A

## Tools Used

Manual review.

## Recommended Mitigation Steps

There are multiple ways to avoid this:

1. Make use of calldata slices and conversions

Something along the lines of (untested!):
```
            // Copy last 20 bytes
            bytes calldata data = msg.data[(msg.data.length - 20):];
            sender = payable(address(uint160(bytes20(data))));
```

2. Implementing purely in assembly

The OpenZeppelin implementation (https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/metatx/ERC2771Context.sol#L21-L30) is an example of an optimised assembly version:
```
            assembly {
                sender := shr(96, calldataload(sub(calldatasize(), 20)))
            }
```

3. Combining slices and assembly

One must note that the pure assembly version is obviously the most gas efficient, at least today.


