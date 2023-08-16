## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed
- disagree with severity

# [Single-step process for critical ownership transfer is risky](https://github.com/code-423n4/2021-06-gro-findings/issues/46) 

# Handle

0xRajeev


# Vulnerability details

## Impact

The Controller contract is arguably the most critical contract in the project for access control management (it has 17 onlyOwner functions). Given that it is derived from Ownable, the ownership management of this contract (also Whitelist and Controllable) defaults to Ownable’s transferOwnership() and renounceOwnership() methods which are not overridden here. Such critical address transfer/renouncing in one-step is very risky because it is irrecoverable from any mistakes.

The same applies to the changing of controller’s address in contracts deriving from Controllable using setController().

Scenario: If an incorrect address, e.g. for which the private key is not known, is used accidentally then it prevents the use of all the onlyOwner() functions forever, which includes the changing of various critical addresses and parameters. This use of incorrect address may not even be immediately apparent given that these functions are probably not used immediately. When noticed, due to a failing onlyOwner() function call, it will force the redeployment of these contracts and require appropriate changes and notifications for switching from the old to new address. This will diminish trust in the protocol and incur a significant reputational damage.


## Proof of Concept

See similar High Risk severity finding from Trail-of-Bits Audit of Hermez: https://github.com/trailofbits/publications/blob/master/reviews/hermez.pdf

See similar Medium Risk severity finding from Trail-of-Bits Audit of Uniswap V3: https://github.com/Uniswap/uniswap-v3-core/blob/main/audits/tob/audit.pdf

https://github.com/OpenZeppelin/openzeppelin-contracts/blob/b9e2c7896d899de9960f2b3d17ca04d5beb79e8a/contracts/access/Ownable.sol#L46-L64

https://github.com/code-423n4/2021-06-gro/blob/091660467fc8d13741f8aafcec80f1e8cf129a33/contracts/Controller.sol#L38

https://github.com/code-423n4/2021-06-gro/blob/091660467fc8d13741f8aafcec80f1e8cf129a33/contracts/Controller.sol#L101

https://github.com/code-423n4/2021-06-gro/blob/091660467fc8d13741f8aafcec80f1e8cf129a33/contracts/Controller.sol#L105

https://github.com/code-423n4/2021-06-gro/blob/091660467fc8d13741f8aafcec80f1e8cf129a33/contracts/Controller.sol#L112

https://github.com/code-423n4/2021-06-gro/blob/091660467fc8d13741f8aafcec80f1e8cf129a33/contracts/Controller.sol#L137

And many other onlyOwner functions such as setController():

https://github.com/code-423n4/2021-06-gro/blob/091660467fc8d13741f8aafcec80f1e8cf129a33/contracts/common/Controllable.sol#L35-L40


## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Override the inherited methods to null functions and use separate functions for a two-step address change: 1) Approve a new address as a pendingOwner 2) A transaction from the pendingOwner address claims the pending ownership change. This mitigates risk because if an incorrect address is used in step (1) then it can be fixed by re-approving the correct address. Only after a correct address is used in step (1) can step (2) happen and complete the address/ownership change.

Also, consider adding a time-delay for such sensitive actions. And at a minimum, use a multisig owner address and not an EOA.

