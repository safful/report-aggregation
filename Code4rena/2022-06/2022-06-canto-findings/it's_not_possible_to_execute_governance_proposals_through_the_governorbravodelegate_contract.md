## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [It's not possible to execute governance proposals through the GovernorBravoDelegate contract](https://github.com/code-423n4/2022-06-canto-findings/issues/39) 

# Lines of code

https://github.com/Plex-Engineer/lending-market/blob/main/contracts/Governance/GovernorBravoDelegate.sol#L63
https://github.com/Plex-Engineer/lending-market/blob/main/contracts/Governance/GovernorBravoDelegate.sol#L87


# Vulnerability details

## Impact
It's not possible to execute a proposal through the GovernorBravoDelegate contract because the `executed` property of it is set to `true` when it's queued up.

Since this means that the governance contract is unusable, it might result in locked-up funds if those were transferred to the contract before the issue comes up. Because of that I'd rate it as HIGH.

## Proof of Concept
`executed` is set to `true`: https://github.com/Plex-Engineer/lending-market/blob/main/contracts/Governance/GovernorBravoDelegate.sol#L63

Here, the `execute()` function checks whether the proposal's state is `Queued`: https://github.com/Plex-Engineer/lending-market/blob/main/contracts/Governance/GovernorBravoDelegate.sol#L87
But, since the `execute` property is `true`, the `state()` function will return `Executed`: https://github.com/Plex-Engineer/lending-market/blob/main/contracts/Governance/GovernorBravoDelegate.sol#L117

In the original compound repo, `executed` is `false` when the proposal is queued up: https://github.com/compound-finance/compound-protocol/blob/master/contracts/Governance/GovernorBravoDelegate.sol#L111

## Tools Used
none

## Recommended Mitigation Steps
Just delete the line where `executed` is set to `true`. Since the zero-value is `false` anyway, you'll save gas as well.

