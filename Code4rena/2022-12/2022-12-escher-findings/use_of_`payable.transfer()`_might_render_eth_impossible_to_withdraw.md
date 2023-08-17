## Tags

- bug
- 2 (Med Risk)
- primary issue
- selected for report
- sponsor confirmed
- edited-by-warden
- M-02

# [Use of `payable.transfer()` Might Render ETH Impossible to Withdraw](https://github.com/code-423n4/2022-12-escher-findings/issues/99) 

# Lines of code

https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol#L105
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol#L85-L86
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPrice.sol#L109
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEdition.sol#L92


# Vulnerability details

## Impact
The protocol uses Solidity’s `transfer()` when transferring ETH to the recipients. This has some notable shortcomings when the recipient is a smart contract, which can render ETH impossible to transfer. Specifically, the transfer will inevitably fail when the smart contract:

- does not implement a payable fallback function, or
- implements a payable fallback function which would incur more than 2300 gas units, or
- implements a payable fallback function incurring less than 2300 gas units but is called through a proxy that raises the call’s gas usage above 2300.

## Proof of Concept
[File: LPDA.sol](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol)

```
85:            ISaleFactory(factory).feeReceiver().transfer(fee);
86:            temp.saleReceiver.transfer(totalSale - fee);

105:        payable(msg.sender).transfer(owed);
```
[File: FixedPrice.sol#L109](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPrice.sol#L109)

```
109:        ISaleFactory(factory).feeReceiver().transfer(address(this).balance / 20);
```
[File: OpenEdition.sol#L92](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEdition.sol#L92)

```
92:        ISaleFactory(factory).feeReceiver().transfer(address(this).balance / 20);
```
Issues pertaining to the use of `transfer()` in the code blocks above may be referenced further via:

- [CONSENSYS Diligence's article](https://consensys.net/diligence/blog/2019/09/stop-using-soliditys-transfer-now/)
- [OpenZeppelin news & events](https://blog.openzeppelin.com/reentrancy-after-istanbul/)
 
## Tools Used
Manual inspection

## Recommended Mitigation Steps
Using `call` with its returned boolean checked in combination with re-entrancy guard is highly recommended after December 2019.

For instance, [line 105](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol#L105) in `LPDA.sol` may be refactored as follows:

```
- payable(msg.sender).transfer(owed);
+ (bool success, ) = payable(msg.sender).call{ value: owed }('');
+ require(success, " Transfer of ETH Failed");
```
Alternatively, `Address.sendValue()` available in [OpenZeppelin Contract’s Address library](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/utils/Address.sol#L44-L65) can be used to transfer the Ether without being limited to 2300 gas units. 

And again, in either of the above measures adopted, the risks of re-entrancy stemming from the use of this function can be mitigated by tightly following the “Check-effects-interactions” pattern and/or using [OpenZeppelin Contract’s ReentrancyGuard contract](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/security/ReentrancyGuard.sol#L43-L54).