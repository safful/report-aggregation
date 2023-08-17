## Tags

- bug
- 2 (Med Risk)
- resolved
- sponsor confirmed
- old-submission-method

# [Put option sellers can prevent exercise by specifying zero amounts, or non-existant tokens](https://github.com/code-423n4/2022-06-putty-findings/issues/223) 

# Lines of code

https://github.com/code-423n4/2022-06-putty/blob/3b6b844bc39e897bd0bbb69897f2deff12dc3893/contracts/src/PuttyV2.sol#L453-L454


# Vulnerability details

## Impact
Put option buyers pay an option premium to the seller for the privilege of being able to 'put' assets to the seller and get the strike price for it rather than the current market price. If they're unable to perform the 'put', they've paid the premium for nothing, and essentially have had funds stolen from them.


## Proof of Concept
If the put option seller includes in `order.erc20Assets`, an amount of zero for any of the assets, or specifies an asset that doesn't currently have any code at its address, the put buyer will be unable to exercise the option, and will have paid the premium for nothing:
```solidity
File: contracts/src/PuttyV2.sol   #1

453               // transfer assets from exerciser to putty
454               _transferERC20sIn(order.erc20Assets, msg.sender);
```
https://github.com/code-423n4/2022-06-putty/blob/3b6b844bc39e897bd0bbb69897f2deff12dc3893/contracts/src/PuttyV2.sol#L453-L454


The function reverts if any amount is equal to zero, or the asset doesn't exist:
```solidity
File: contracts/src/PuttyV2.sol   #2

593       function _transferERC20sIn(ERC20Asset[] memory assets, address from) internal {
594           for (uint256 i = 0; i < assets.length; i++) {
595               address token = assets[i].token;
596               uint256 tokenAmount = assets[i].tokenAmount;
597   
598               require(token.code.length > 0, "ERC20: Token is not contract");
599               require(tokenAmount > 0, "ERC20: Amount too small");
600   
601               ERC20(token).safeTransferFrom(from, address(this), tokenAmount);
602           }
603       }
```
https://github.com/code-423n4/2022-06-putty/blob/3b6b844bc39e897bd0bbb69897f2deff12dc3893/contracts/src/PuttyV2.sol#L593-L603



## Tools Used
Code inspection


## Recommended Mitigation Steps
Verify the asset amounts and addresses during `fillOrder()`, and allow exercise if the token no longer exists at that point in time



