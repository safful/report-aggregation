## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed

# [confusing comment in protocolUpdate](https://github.com/code-423n4/2021-07-sherlock-findings/issues/19) 

# Handle

gpersoon


# Vulnerability details

## Impact
The comment "NOTE: UNUSED" can be interpreted that both protocolManagers and protocolAgents are unused. See proof of concept below.
However only protocolManagers is unused. 

## Proof of Concept
//https://github.com/code-423n4/2021-07-sherlock/blob/main/contracts/facets/Gov.sol#L148
function protocolUpdate( bytes32 _protocol, address _eoaProtocolAgent,address _eoaManager) public override onlyGovMain {
...
    // NOTE: UNUSED
    gs.protocolManagers[_protocol] = _eoaManager;
    gs.protocolAgents[_protocol] = _eoaProtocolAgent;
  }

## Tools Used

## Recommended Mitigation Steps
Change the comment to:
// NOTE: protocolManagers UNUSED



