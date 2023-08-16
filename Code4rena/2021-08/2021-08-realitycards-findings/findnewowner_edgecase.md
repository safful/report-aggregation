## Tags

- bug
- sponsor confirmed
- Resolved
- disagree with severity
- 3 (High Risk)

# [findNewOwner edgecase](https://github.com/code-423n4/2021-08-realitycards-findings/issues/27) 

# Handle

gpersoon


# Vulnerability details

## Impact
In the function findNewOwner of RCOrderbook, as loop is done which included the check  _loopCounter < maxDeletions
Afterwards a check is done for  "(_loopCounter != maxDeletions)" to determine if the processing is finished.
If _loopCounter == maxDeletions then the conclusion is that it isn't finished yet.

However there is the edgecase that the processing might just be finished at the same time as _loopCounter == maxDeletions.

You can see this the best if you assume maxDeletions==1, in that case it will never draw the conclusion it is finished.
Of course having maxDeletions==1 is very unlikely in practice.

## Proof of Concept
// https://github.com/code-423n4/2021-08-realitycards/blob/main/contracts/RCOrderbook.sol#L549
 function findNewOwner(uint256 _card, uint256 _timeOwnershipChanged)  external  override  onlyMarkets  {
...
        // delete current owner
        do {
            _newPrice = _removeBidFromOrderbookIgnoreOwner( _head.next, _market, _card );
            _loopCounter++;             // delete next bid if foreclosed
        } while (    treasury.foreclosureTimeUser( _head.next, _newPrice,  _timeOwnershipChanged ) <  minimumTimeToOwnTo &&
                _loopCounter < maxDeletions );

        if (_loopCounter != maxDeletions) {   // the old owner is dead, long live the new owner
            _newOwner = .... 
            ...
        } else {
            // we hit the limit, save the old owner, we'll try again next time
           ...
        }
    }


## Tools Used

## Recommended Mitigation Steps
Use a different way to determine that the processing is done. This could save some gas.
Note: the additional check also costs gas, so you have the verify the end result.

Perhaps in setDeletionLimit doublecheck that _deletionLimit > 1.

