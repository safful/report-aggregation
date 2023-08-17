## Tags

- bug
- 2 (Med Risk)
- primary issue
- satisfactory
- sponsor confirmed
- selected for report
- M-10

# [address.call{value:x}() should be used instead of payable.transfer()](https://github.com/code-423n4/2022-11-debtdao-findings/issues/369) 

# Lines of code

https://github.com/debtdao/Line-of-Credit/blob/e8aa08b44f6132a5ed901f8daa231700c5afeb3a/contracts/utils/LineLib.sol#L48


# Vulnerability details

## Impact

When withdrawing and refund  ETH, the  contract uses Solidity’s `transfer()` function. 

Using Solidity's `transfer()` function has some notable shortcomings when the withdrawer is a smart contract, which can render ETH deposits impossible to withdraw. Specifically, the withdrawal will inevitably fail when:
* The withdrawer smart contract does not implement a payable fallback function.
* The withdrawer smart contract implements a payable fallback function which uses more than 2300 gas units.
* The withdrawer smart contract implements a payable fallback function which needs less than 2300 gas units but is called through a proxy that raises the call’s gas usage above 2300.

Risks of reentrancy stemming from the use of this function can be mitigated by tightly following the "Check-Effects-Interactions" pattern and using OpenZeppelin Contract’s ReentrancyGuard contract. 

## Proof of Concept

```solidity
// Line-of-Credit/contracts/utils/LineLib.sol
48:    payable(receiver).transfer(amount);
```


#### References:

The issues with `transfer()` are outlined [here](https://consensys.net/diligence/blog/2019/09/stop-using-soliditys-transfer-now/
)

For further reference on why using Solidity’s `transfer()` is no longer recommended, refer to these [articles](https://blog.openzeppelin.com/reentrancy-after-istanbul/).



## Tools Used
Manual analysis.

## Recommended Mitigation Steps

Using low-level `call.value(amount)` with the corresponding result check or using the OpenZeppelin `Address.sendValue` is advised, [reference](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/utils/Address.sol#L60).


