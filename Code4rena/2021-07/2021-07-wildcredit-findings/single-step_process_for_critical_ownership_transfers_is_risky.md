## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed

# [Single-step process for critical ownership transfers is risky](https://github.com/code-423n4/2021-07-wildcredit-findings/issues/82) 

# Handle

0xRajeev


# Vulnerability details

## Impact

Multiple contracts: Controller, LPTokenMaster, RewardDistribution and UniswapV3Oracle use onlyOwner authorized functions. Given that this is derived from Ownable, the ownership management of these contracts defaults to Ownable’s transferOwnership() and renounceOwnership() methods which are not overridden here. Such critical address transfer/renouncing in one-step is very risky because it is irrecoverable from any mistakes.

Scenario: If an incorrect address, e.g. for which the private key is not known, is used accidentally then it prevents the use of all the onlyOwner() functions forever, which includes the changing of various critical addresses and parameters. This use of incorrect address may not even be immediately apparent given that these functions are probably not used immediately. When noticed, due to a failing onlyOwner() function call, it will force the redeployment of these contracts and require appropriate changes and notifications for switching from the old to new address. This will diminish trust in the protocol and incur a significant reputational damage.


## Proof of Concept

See similar High Risk severity finding from Trail-of-Bits Audit of Hermez: https://github.com/trailofbits/publications/blob/master/reviews/hermez.pdf

See similar Medium Risk severity finding from Trail-of-Bits Audit of Uniswap V3: https://github.com/Uniswap/uniswap-v3-core/blob/main/audits/tob/audit.pdf

https://github.com/code-423n4/2021-07-wildcredit/blob/82c48d73fd27a9d4d5d4a395b3affcef4ef6c5c8/contracts/external/Ownable.sol#L25-L38

https://github.com/code-423n4/2021-07-wildcredit/blob/82c48d73fd27a9d4d5d4a395b3affcef4ef6c5c8/contracts/Controller.sol#L11

https://github.com/code-423n4/2021-07-wildcredit/blob/82c48d73fd27a9d4d5d4a395b3affcef4ef6c5c8/contracts/LPTokenMaster.sol#L12

https://github.com/code-423n4/2021-07-wildcredit/blob/82c48d73fd27a9d4d5d4a395b3affcef4ef6c5c8/contracts/RewardDistribution.sol#L17

https://github.com/code-423n4/2021-07-wildcredit/blob/82c48d73fd27a9d4d5d4a395b3affcef4ef6c5c8/contracts/UniswapV3Oracle.sol#L12

## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Override the inherited methods to null functions and use separate functions for a two-step address change: 1) Approve a new address as a pendingOwner 2) A transaction from the pendingOwner address claims the pending ownership change. This mitigates risk because if an incorrect address is used in step (1) then it can be fixed by re-approving the correct address. Only after a correct address is used in step (1) can step (2) happen and complete the address/ownership change.

Also, consider adding a timelock delay for such sensitive actions. And at a minimum, use a multisig (with mutually independent and trustworthy owners) and not an EOA.

