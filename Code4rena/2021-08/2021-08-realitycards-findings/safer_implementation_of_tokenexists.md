## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed
- Resolved

# [safer implementation of tokenExists](https://github.com/code-423n4/2021-08-realitycards-findings/issues/8) 

# Handle

gpersoon


# Vulnerability details

## Impact
The function tokenExists does only limited checks on the existence of cards.
It doesn't doublecheck that tokenIds[_card] != 0  
This is relevant because 0 is the default value of empty array elements. Although this isn't a problem in the current code, 
future changes might accidentally introduce vulnerabilities.

Also cards are only valid if they are below numberOfCards. This has led to vulnerabilities in previous versions of the contract
(e.g. previous contest)

## Proof of Concept
// https://github.com/code-423n4/2021-08-realitycards/blob/main/contracts/RCMarket.sol#L1139
function tokenExists(uint256 _card) internal view returns (bool) {
        return tokenIds[_card] != type(uint256).max;
}

## Tools Used

## Recommended Mitigation Steps
Change the function to something like the following:

function tokenExists(uint256 _card) internal view returns (bool) {
       if (_cardId >= numberOfCards) return false;
       if (tokenIds[_card] == 0) return false;
       return tokenIds[_card] != type(uint256).max;
}


