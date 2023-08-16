## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed
- Resolved

# [Can't retrieve all data with getMarketInfo ](https://github.com/code-423n4/2021-08-realitycards-findings/issues/14) 

# Handle

gpersoon


# Vulnerability details

## Impact
The function getMarketInfo of RCFactory only can give results back in the range 0...marketInfoResults 
Supplying _skipResults doesn't help, it then just skips the first _skipResults  records.

Assume marketInfoResults == 10 and _skipResults == 20:
Then no result will be given back because "_resultNumber < marketInfoResults" will never allow _resultNumber  to be bigger than 10

Note: this is low risk because getMarketInfo is a backup function (although you maybe want the backup to function as expected)

## Proof of Concept
// https://github.com/code-423n4/2021-08-realitycards/blob/main/contracts/RCFactory.sol#L227
  function getMarketInfo( IRCMarket.Mode _mode, uint256 _state, uint256 _skipResults  )  external view
        returns ( address[] memory, string[] memory, string[] memory, uint256[] memory ) {   
        ..
        uint256 _resultNumber = 0;
       ..
        while (_resultNumber < marketInfoResults && _marketIndex > 1) {
           ...
                if (_resultNumber < _skipResults) {
                    _resultNumber++;
                } else {
                    _marketAddresses[_resultNumber] = _market;   // will never reach this part if _skipResults >= marketInfoResults 
                    ....
                    _resultNumber++;
                }
            }
        }
        return (_marketAddresses, _ipfsHashes, _slugs, _potSizes);
    }


## Tools Used

## Recommended Mitigation Steps
Update the code to something like the following:
    
 uint idx;
 while (idx < marketInfoResults && _marketIndex > 1) {
            _marketIndex--;
            address _market = marketAddresses[_mode][_marketIndex];
            if (IRCMarket(_market).state() == IRCMarket.States(_state)) {
                if (_resultNumber < _skipResults) {
                    _resultNumber++;
                } else {
                    _marketAddresses[idx] = _market;
                    _ipfsHashes[idx] = ipfsHash[_market];
                    _slugs[idx] = addressToSlug[_market];
                    _potSizes[idx] = IRCMarket(_market).totalRentCollected();
                    idx++;
                }
            }
        }    



