## Tags

- bug
- 2 (Med Risk)
- disagree with severity
- resolved
- sponsor confirmed
- selected-for-report

# [GolomToken.sol doesn't contain a function to mint treasury tokens](https://github.com/code-423n4/2022-07-golom-findings/issues/205) 

# Lines of code

https://github.com/code-423n4/2022-07-golom/blob/7bbb55fca61e6bae29e57133c1e45806cbb17aa4/contracts/governance/GolomToken.sol#L14-L73


# Vulnerability details

## Impact
Potential downtime in GolomTrader

## Proof of Concept
GolomToken.sol doesn't have a function to mint the treasury tokens as specified in the docs (https://docs.golom.io/tokenomics-and-airdrop). In order for these tokens to be minted, the minter would have to be changed via setMinter() and executeSetMinter() to a contract that can mint the treasury tokens. Because of the 24 hour timelock, this would lead to downtime for GolomTrader.sol if trading has already begun. This is because GolomTrader.sol calls RewardDistributor.sol#addFees each time there is a filled order. When the epoch changes, RewardDistributor.sol will try to call the mint function in GolomToken.sol. Because of the timelock, there will be at least a 24 hours period where RewardDistributor.sol is not the minter and doesn't have the permission to mint. This means that during that period all trades will revert.

## Tools Used

## Recommended Mitigation Steps
Add a function to GolomToken.sol to mint the treasury tokens similar to the mintAirdrop() and mintGenesisReward() functions.