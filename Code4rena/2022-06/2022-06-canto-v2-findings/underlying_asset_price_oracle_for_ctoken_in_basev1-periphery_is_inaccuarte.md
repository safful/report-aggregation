## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [Underlying asset price oracle for CToken in BaseV1-periphery is inaccuarte](https://github.com/code-423n4/2022-06-canto-v2-findings/issues/134) 

# Lines of code

https://github.com/Plex-Engineer/lending-market-v2/blob/443a8c0fed3c5018e95f3881a31b81a555c42b2d/contracts/Stableswap/BaseV1-periphery.sol#L489


# Vulnerability details

## Impact
Detailed description of the impact of this finding.

Underlying asset price oracle for CToken in BaseV1-periphery is inaccuarte

## Proof of Concept
Provide direct links to all referenced code in GitHub. Add screenshots, logs, or any other relevant proof that illustrates the concept.

```
    function getUnderlyingPrice(CToken ctoken) external override view returns(uint price) {
        IBaseV1Pair pair;
        uint8 stable;
        bool stablePair;
        address underlying;

        if (compareStrings(ctoken.symbol(), "cCANTO")) {
            stable = 0;
            underlying = address(wcanto);
        } 
        //set price statically to 1 when the Comptroller is retrieving Price
        else if (compareStrings(ctoken.symbol(), "cNOTE") && msg.sender == Comptroller) {
            return 1; // Note price is fixed to 1
        }
```

we should not be return 1. 1 is 1 wei. we should be 10 ** 18

## Tools Used
VIM

## Recommended Mitigation Steps

we can return 10 ** 18

