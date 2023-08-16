## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed
- resolved

# [Use immutable keyword](https://github.com/code-423n4/2021-06-realitycards-findings/issues/6) 

# Handle

gpersoon


# Vulnerability details

## Impact
Several variables are only set once. So it might be useful to make them immutable to reduced the change of accidental updates
See the "proof of concept" for examples.

## Proof of Concept
// https://github.com/code-423n4/2021-06-realitycards/blob/main/contracts/RCMarket.sol
uint256 public numberOfCards;
uint256 public totalNftMintCount;
IRCTreasury public treasury;
IRCFactory public factory;
IRCNftHubL2 public nfthub;
IRCOrderbook public orderbook;
uint256 public minimumPriceIncreasePercent;
uint256 public minRentalDayDivisor;
uint256 public maxRentIterations;
uint32 public marketOpeningTime;
uint32 public override marketLockingTime;
uint32 public oracleResolutionTime;   
address public artistAddress;
address public affiliateAddress;
address public marketCreatorAddress;
uint256 public creatorCut;    
address[] public cardAffiliateAddresses;
address public arbitrator;
IRealitio public realitio;

//https://github.com/code-423n4/2021-06-realitycards/blob/main/contracts/RCFactory.sol#L113
IRCTreasury public override treasury;
    
## Tools Used

## Recommended Mitigation Steps
Use the immutable keyword where possible


