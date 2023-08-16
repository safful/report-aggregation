## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [DOMAIN_SEPARATOR can change](https://github.com/code-423n4/2021-11-malt-findings/issues/349) 

# Handle

pauliax


# Vulnerability details

## Impact
The variable DOMAIN_SEPARATOR in contract ERC20Permit is assigned in the constructor and will not change after being initialized. However, if a hard fork happens after the contract deployment, the domain would become invalid on one of the forked chains due to the block.chainid has changed. 
Also, you don't need an assmebly to retrieve chainid, you can get it from a built in variable block.chainid.

Similar issues were reported in a previous contest and were assigned a severity of low: 
https://github.com/code-423n4/2021-06-realitycards-findings/issues/166 
https://github.com/code-423n4/2021-09-swivel-findings/issues/98

## Recommended Mitigation Steps
An elegant solution that you may consider applying is from Sushi Trident: https://github.com/sushiswap/trident/blob/concentrated/contracts/pool/concentrated/TridentNFT.sol#L47-L62

