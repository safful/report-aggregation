## Tags

- bug
- sponsor confirmed
- 1 (Low Risk)
- resolved

# [block.chainid may change in case of a hardfork](https://github.com/code-423n4/2021-10-ambire-findings/issues/55) 

# Handle

pauliax


# Vulnerability details

## Impact
The 'DOMAIN_SEPARATOR' is not recalculated in the case of a hard fork. The variable DOMAIN_SEPARATOR in contract QuickAccManager is cached in the contract storage and will not change after being initialized. However, if a hard fork happens after the contract deployment, the domain would become invalid on one of the forked chains due to the block.chainid has changed. 
A similar issue was reported in a previous contest and was assigned a severity of low: https://github.com/code-423n4/2021-06-realitycards-findings/issues/166

## Recommended Mitigation Steps
An elegant solution that you may consider applying is from Sushi Trident: https://github.com/sushiswap/trident/blob/concentrated/contracts/pool/concentrated/TridentNFT.sol#L47-L62

