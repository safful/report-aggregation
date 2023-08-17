## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [RubiconRouter: Offers created through offerWithETH() can be cancelled by anyone](https://github.com/code-423n4/2022-05-rubicon-findings/issues/17) 

# Lines of code

https://github.com/code-423n4/2022-05-rubicon/blob/8c312a63a91193c6a192a9aab44ff980fbfd7741/contracts/RubiconRouter.sol#L383-L409


# Vulnerability details

## Impact
When a user creates an offer through the offerWithETH function of the RubiconRouter contract, the offer function of the RubiconMarket contract is called, and the RubiconRouter contract address is set to offer.owner in the offer function.
This means that anyone can call the cancelForETH function of the RubiconRouter contract to cancel the offer and get the ether.
## Proof of Concept
https://github.com/code-423n4/2022-05-rubicon/blob/8c312a63a91193c6a192a9aab44ff980fbfd7741/contracts/RubiconRouter.sol#L383-L409
https://github.com/code-423n4/2022-05-rubicon/blob/8c312a63a91193c6a192a9aab44ff980fbfd7741/contracts/RubiconRouter.sol#L440-L452
## Tools Used
None
## Recommended Mitigation Steps
Set the owner of offer_id to msg.sender in offerWithETH function and check it in cancelForETH function

