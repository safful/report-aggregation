## Tags

- bug
- 0 (Non-critical)
- resolved
- sponsor confirmed

# [`transfer()` is not recommended for sending ETH](https://github.com/code-423n4/2021-12-nftx-findings/issues/94) 

# Handle

WatchPug


# Vulnerability details

Since the introduction of `transfer()`, it has typically been recommended by the security community because it helps guard against reentrancy attacks. This guidance made sense under the assumption that gas costs wouldn’t change. It's now recommended that transfer() and send() be avoided, as gas costs can and will change and reentrancy guard is more commonly used.

Any smart contract that uses `transfer()` is taking a hard dependency on gas costs by forwarding a fixed amount of gas: 2300.

It's recommended to stop using `transfer()` and switch to using `call()` instead.

https://github.com/code-423n4/2021-12-nftx/blob/194073f750b7e2c9a886ece34b6382b4f1355f36/nftx-protocol-v2/contracts/solidity/NFTXV1Buyout.sol#L44-L44

```solidity
payable(msg.sender).transfer(amount);
```

Can be changed to:

```solidity
(bool success, ) = msg.sender.call{value: amount}("");
require(success, "ETH transfer failed");
```

