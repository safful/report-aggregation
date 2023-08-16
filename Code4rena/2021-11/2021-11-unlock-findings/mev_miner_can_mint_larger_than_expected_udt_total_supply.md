## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [MEV miner can mint larger than expected UDT total supply](https://github.com/code-423n4/2021-11-unlock-findings/issues/135) 

# Handle

elprofesor


# Vulnerability details

## Impact
`UnlockProtocol` attempts to calculate gas reimbursement using tx.gasprice, typically users who falsify tx.gasprice would lose gas to miners and therefore not obtain any advantage over the protocol itself. This does present capabilities for miners to extract value, as they can submit their own transactions, or cooperate with a malicious user, reimbursing a portion (or all) or the tx.gasprice used. As the following calculation is made;
```
    uint tokensToDistribute = (estimatedGasForPurchase * tx.gasprice) * (125 * 10 ** 18) / 100 / udtPrice;
```

we can see that arbitrary tx.gasprices can rapidly inflate the `tokensToDistribute`. Though capped at maxTokens, this value can be up to half the total supply of UDT, which could dramatically affect the value of UDT potentially leading to lucrative value extractions outside of the pool.

## Proof of Concept

## Recommended Mitigation Steps
Using an oracle service to determine the average gas price and ensuring it is within some normal bounds that has not been subjected to arbitrary value manipulation.

