## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [RubiconRouter: Excess ether did not return to the user](https://github.com/code-423n4/2022-05-rubicon-findings/issues/15) 

# Lines of code

https://github.com/code-423n4/2022-05-rubicon/blob/8c312a63a91193c6a192a9aab44ff980fbfd7741/contracts/RubiconRouter.sol#L325-L339


# Vulnerability details

## Impact
In  swapWithETH/buyAllAmountWithETH/offerWithETH/depositWithETH functions of the RubiconRouter contract, when msg.value > max_fill_withFee/pay_amt/amount/amtWithFee, the excess ether will not be returned to the user.

## Proof of Concept
https://github.com/code-423n4/2022-05-rubicon/blob/8c312a63a91193c6a192a9aab44ff980fbfd7741/contracts/RubiconRouter.sol#L325-L339
https://github.com/code-423n4/2022-05-rubicon/blob/8c312a63a91193c6a192a9aab44ff980fbfd7741/contracts/RubiconRouter.sol#L383-L393
https://github.com/code-423n4/2022-05-rubicon/blob/8c312a63a91193c6a192a9aab44ff980fbfd7741/contracts/RubiconRouter.sol#L455-L462
https://github.com/code-423n4/2022-05-rubicon/blob/8c312a63a91193c6a192a9aab44ff980fbfd7741/contracts/RubiconRouter.sol#L494-L507
## Tools Used
None
## Recommended Mitigation Steps
Return excess ether to msg.sender, or require msg.value == max_fill_withFee/pay_amt/amount/amtWithFee

