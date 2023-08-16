## Tags

- bug
- 2 (Med Risk)
- resolved
- sponsor confirmed

# [Arbitrary code can be run with Controller as msg.sender](https://github.com/code-423n4/2022-03-rolla-findings/issues/65) 

# Lines of code

https://github.com/code-423n4/2022-03-rolla/blob/main/quant-protocol/contracts/Controller.sol#L497-L516


# Vulnerability details

## Impact

A malicious user can call Controller's operate with ActionType.QTokenPermit, providing a precooked contract address as qToken, that will be called by Controller contract with IQToken(_qToken).permit(), which implementation can be arbitrary as long as IQToken interface and permit signature is implemented.

The Controller is asset bearing contract and it will be msg.sender in this arbitrary permit() function called, which is a setup that better be avoided.

## Proof of Concept

When the Controller's operate with a QTokenPermit action, it parses the arguments with Actions library and then calls internal _qTokenPermit:

https://github.com/code-423n4/2022-03-rolla/blob/main/quant-protocol/contracts/Controller.sol#L91-L92

_qTokenPermit calls the IQToken(_qToken) address provided without performing any additional checks:

https://github.com/code-423n4/2022-03-rolla/blob/main/quant-protocol/contracts/Controller.sol#L497-L516

This way, contrary to the approach used in other actions, qToken isn't checked to be properly created address and is used right away, while the requirement that the address provided should implement IQToken interface and have permit function with a given signature can be easily met with a precooked contract.

## Recommended Mitigation Steps

Given that QToken can be called directly please examine the need for QTokenPermit ActionType.

If current approach is based on UI convenience and better be kept, consider probing for IOptionsFactory(optionsFactory).isQToken(_qToken) before calling the address provided.

