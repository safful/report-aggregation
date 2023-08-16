## Tags

- bug
- 2 (Med Risk)
- resolved
- sponsor confirmed

# [Spreads can be minted with a deactivated oracle](https://github.com/code-423n4/2022-03-rolla-findings/issues/66) 

# Lines of code

https://github.com/code-423n4/2022-03-rolla/blob/main/quant-protocol/contracts/libraries/FundsCalculator.sol#L91-L117


# Vulnerability details

## Impact

When deactivateOracle() is called for an oracle in OracleRegistry it is still available for option spreads minting.

This way a user can continue to mint new options within spreads that rely on an oracle that was deactivated. As economic output of spreads is close to vanilla options, so all users who already posses an option linked to a deactivated oracle can surpass this deactivation, being able to mint new options linked to it as a part of option spreads.

## Proof of Concept

Oracle active state is checked with isOracleActive() during option creation in validateOptionParameters() and during option minting in _mintOptionsPosition().

It isn't checked during spreads creation:

https://github.com/code-423n4/2022-03-rolla/blob/main/quant-protocol/contracts/libraries/FundsCalculator.sol#L91-L117

In other words besides vanilla option minting and creation all spectrum of operations is available for the deactivated oracle assets, including spreads minting, which economically is reasonably close to vanilla minting.

## Recommended Mitigation Steps

If oracle deactivation is meant to transfer all related assets to the close only state then consider requiring oracle to be active on spreads minting as well in the same way it's done for vanilla option minting:

https://github.com/code-423n4/2022-03-rolla/blob/main/quant-protocol/contracts/Controller.sol#L188-L197


