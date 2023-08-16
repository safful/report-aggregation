## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed
- Resolved

# [make code of rentAllCards easier to read and maintain](https://github.com/code-423n4/2021-08-realitycards-findings/issues/20) 

# Handle

gpersoon


# Vulnerability details

## Impact
The function rentAllCards or RCMarket contains a similar piece of code twice
(mainly the formula: (card[i].cardPrice * (minimumPriceIncreasePercent + 100)) / 100; )

The current code is somewhat difficult to read and maintain and hides potential issue (also see other issue about rentAllCards)

## Proof of Concept
//https://github.com/code-423n4/2021-08-realitycards/blob/main/contracts/RCMarket.sol#L691
  function rentAllCards(uint256 _maxSumOfPrices) external override {
        _checkState(States.OPEN);
        // check that not being front run
        uint256 _actualSumOfPrices = 0;
        for (uint256 i = 0; i < numberOfCards; i++) {
            if (card[i].cardPrice == 0) {
                _actualSumOfPrices += MIN_RENTAL_VALUE;
            } else {
                _actualSumOfPrices += (card[i].cardPrice * (minimumPriceIncreasePercent + 100)) / 100;
            }
        }
        require(_actualSumOfPrices <= _maxSumOfPrices, "Prices too high");

        for (uint256 i = 0; i < numberOfCards; i++) {
            if (ownerOf(i) != msgSender()) {
                uint256 _newPrice;
                if (card[i].cardPrice > 0) {
                    _newPrice =
                        (card[i].cardPrice *
                            (minimumPriceIncreasePercent + 100)) /
                        100;
                } else {
                    _newPrice = MIN_RENTAL_VALUE;
                }
                newRental(_newPrice, 0, address(0), i);
            }
        }
    }


## Tools Used

## Recommended Mitigation Steps
Suggestion to make the code easier to read and maintain:

 function calc(uint256 currentPrice) returns(uint256) {
        if (currentPrice == 0) 
            return MIN_RENTAL_VALUE;
        return (currentPrice *(minimumPriceIncreasePercent + 100)) / 100;
    }
    
    function rentAllCards(uint256 _maxSumOfPrices) external override {
      ..
        uint256 _actualSumOfPrices = 0;
        for (uint256 i = 0; i < numberOfCards; i++) {
            _actualSumOfPrices += calc(card[i].cardPrice);
        }
        .....
        for (uint256 i = 0; i < numberOfCards; i++) {
            if (ownerOf(i) != msgSender()) {
                uint256 _newPrice=calc(card[i].cardPrice);
                newRental(_newPrice, 0, address(0), i);
            }
        }
    }


