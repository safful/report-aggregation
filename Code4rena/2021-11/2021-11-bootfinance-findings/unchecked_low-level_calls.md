## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Unchecked low-level calls](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/145) 

# Handle

0v3rf10w


# Vulnerability details

## Impact
Unchecked low-level calls

## Proof of Concept
Unchecked cases at 2 places :-
BasicSale.receive() (2021-11-bootfinance/tge/contracts/PublicSale.sol#148-156) ignores return value by burnAddress.call{value: msg.value}() (2021-11-bootfinance/tge/contracts/PublicSale.sol#154)

BasicSale.burnEtherForMember(address) (2021-11-bootfinance/tge/contracts/PublicSale.sol#158-166) ignores return value by burnAddress.call{value: msg.value}() (2021-11-bootfinance/tge/contracts/PublicSale.sol#164)


## Tools Used
Manual

## Recommended Mitigation Steps
The return value of the low-level call is not checked, so if the call fails, the Ether will be locked in the contract. If the low level is used to prevent blocking operations, consider logging failed calls.

