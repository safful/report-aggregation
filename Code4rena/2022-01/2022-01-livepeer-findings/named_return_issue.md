## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Named return issue](https://github.com/code-423n4/2022-01-livepeer-findings/issues/6) 

# Handle

robee


# Vulnerability details

Users can mistakenly think that the return value is the named return, but it is actually the actualreturn statement that comes after. To know that the user needs to read the code and is confusing.
Furthermore, removing either the actual return or the named return will save gas. 

        L1LPTGateway.sol, getOutboundCalldata
        L2LPTGateway.sol, outboundTransfer
        L2LPTGateway.sol, getOutboundCalldata

