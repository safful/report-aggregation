## Tags

- bug
- QA (Quality Assurance)
- sponsor confirmed

# [NibblVault: BuyoutInitiated event parameter error](https://github.com/code-423n4/2022-06-nibbl-findings/issues/160) 

# Lines of code

https://github.com/code-423n4/2022-06-nibbl/blob/8c3dbd6adf350f35c58b31723d42117765644110/contracts/NibblVault.sol#L414-L417


# Vulnerability details

## Impact
In the initiateBuyout function of the NibblVault contract, the second parameter of the BuyoutInitiated event is _buyoutBid instead of _currentValuation, and since the excess Ether in _buyoutBid is transferred to the user, the actual buyout price for the user is the _currentValuation variable.
The user can use a large amount of Ether to get a large _buyoutBid variable, however the actual amount of Ether spent by the user is _currentValuation.
Events emitted by the smart contract are used off-chain, and incorrect event parameters may have an impact on the user's trading behavior

## Proof of Concept
https://github.com/code-423n4/2022-06-nibbl/blob/8c3dbd6adf350f35c58b31723d42117765644110/contracts/NibblVault.sol#L414-L417
## Tools Used
None
## Recommended Mitigation Steps
```
-       emit BuyoutInitiated(msg.sender, _buyoutBid);
+      emit BuyoutInitiated(msg.sender, _currentValuation);
```

