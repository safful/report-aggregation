## Tags

- bug
- 2 (Med Risk)
- resolved
- sponsor confirmed
- edited-by-warden

# [The settledPrice maybe exceed maximumPrice](https://github.com/code-423n4/2022-09-party-findings/issues/201) 

# Lines of code

https://github.com/PartyDAO/party-contracts-c4/blob/3896577b8f0fa16cba129dc2867aba786b730c1b/contracts/crowdfund/BuyCrowdfundBase.sol#L122


# Vulnerability details

## Impact

BuyCrowdfundBase.sol _buy()
When callValue = 0 is settledPrice to totalContributions ignoring whether totalContributions > maximumPrice
resulting in the minimum proportion of participants expected to become smaller

## Proof of Concept
```
    function _buy(
        IERC721 token,
        uint256 tokenId,
        address payable callTarget,
        uint96 callValue,
        bytes calldata callData,
        FixedGovernanceOpts memory governanceOpts
    )
    ...
            settledPrice_ = callValue == 0 ? totalContributions : callValue;  //**** not check totalContributions>maximumPrice****//
            if (settledPrice_ == 0) {
                // Still zero, which means no contributions.
                revert NoContributionsError();
            }
            settledPrice = settledPrice_;    
```

(AuctionCrowdfund.sol finalize()  similar)

## Recommended Mitigation Steps
add check

```
    function _buy(
        IERC721 token,
        uint256 tokenId,
        address payable callTarget,
        uint96 callValue,
        bytes calldata callData,
        FixedGovernanceOpts memory governanceOpts
    )
    ...
            settledPrice_ = callValue == 0 ? totalContributions : callValue;
            if (settledPrice_ == 0) {
                // Still zero, which means no contributions.
                revert NoContributionsError();
            }

+++         if (maximumPrice_ != 0 && settledPrice_ > maximumPrice_) {
+++                settledPrice_ = maximumPrice_;
+++         }

            settledPrice = settledPrice_;    
```
