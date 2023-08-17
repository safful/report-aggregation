## Tags

- bug
- 2 (Med Risk)
- resolved
- sponsor confirmed
- selected-for-report

# [Use `call()` rather than `transfer()` on address payable](https://github.com/code-423n4/2022-07-golom-findings/issues/343) 

# Lines of code

https://github.com/code-423n4/2022-07-golom/blob/main/contracts/core/GolomTrader.sol#L154


# Vulnerability details

### Impact

[L154](https://github.com/code-423n4/2022-07-golom/blob/main/contracts/core/GolomTrader.sol#L154) in [GolomTrader.sol](https://github.com/code-423n4/2022-07-golom/blob/main/contracts/core/GolomTrader.sol) uses `.transfer()` to send ether to other addresses. There are a number of issues with using `.transfer()`, as it can fail for a number of reasons (specified in the Proof of Concept).

### Proof of Concept

1. The destination is a smart contract that doesn’t implement a `payable` function or it implements a `payable` function but that function uses more than 2300 gas units.
2. The destination is a smart contract that doesn’t implement a `payable` `fallback` function or it implements a `payable` `fallback` function but that function uses more than 2300 gas units.
3. The destination is a smart contract but that smart contract is called via an intermediate proxy contract increasing the case requirements to more than 2300 gas units. A further example of unknown destination complexity is that of a multisig wallet that as part of its operation uses more than 2300 gas units.
4. Future changes or forks in Ethereum result in higher gas fees than transfer provides. The `.transfer()` creates a hard dependency on 2300 gas units being appropriate now and into the future.

### Tools Used

Vim

### Recommended Remediation Steps

Instead use the `.call()` function to transfer ether and avoid some of the limitations of `.transfer()`. This would be accomplished by changing `payEther()` to something like;

```solidity
(bool success, ) = payable(payAddress).call{value: payAmt}(""); // royalty transfer to royaltyaddress
require(success, "Transfer failed.");
```

Gas units can also be passed to the `.call()` function as a variable to accomodate any uses edge cases. Gas could be a mutable state variable that can be set by the contract owner.
