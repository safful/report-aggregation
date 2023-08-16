## Tags

- bug
- 1 (Low Risk)
- disagree with severity
- sponsor confirmed

# [LaunchEvent pays out fewer incentives then expected](https://github.com/code-423n4/2022-01-trader-joe-findings/issues/82) 

# Handle

TomFrenchBlockchain


# Vulnerability details

## Impact

LaunchEvent pays out fewer incentives than expected.

## Proof of Concept

When creating a launch event, issuers must provide the total amount of tokens they want to send to the contract and what percentage of these are reserved for incentives.

https://github.com/code-423n4/2022-01-trader-joe/blob/a1579f6453bc4bf9fb0db9c627beaa41135438ed/contracts/RocketJoeFactory.sol#L84-L86

Note that there's an inconsistency between the documentation and the implementation. Documentation implies that the issuer provides `_tokenAmount` tokens and an additional `_tokenAmount * _tokenIncentivesPercent / 1e18` as an incentive whereas in reality they only provide `_tokenAmount` 

This can be seen here:

https://github.com/code-423n4/2022-01-trader-joe/blob/a1579f6453bc4bf9fb0db9c627beaa41135438ed/contracts/RocketJoeFactory.sol#L133

For us to pay out the correct percentage of `_tokenAmount` as incentives would expect that amount to be `(_tokenAmount * _tokenIncentivesPercent) / 1e18` however as can be seen we pay out `_tokenAmount - (_tokenAmount * 1e18) / (1e18 + _tokenIncentivesPercent)`

https://github.com/code-423n4/2022-01-trader-joe/blob/a1579f6453bc4bf9fb0db9c627beaa41135438ed/contracts/LaunchEvent.sol#L273

This will consistently pay out a smaller percentage of the total amount of tokens than `_tokenIncentivesPercent`.

## Recommended Mitigation Steps

Switch to having the issuer provide explicit amounts `_tokenIssuanceAmount` and `_tokenIncentivesAmount` to avoid mistakes about how percentages are handled.

Add tests to ensure that the contract is initialised with the correct state.

