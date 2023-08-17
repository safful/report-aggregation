## Tags

- bug
- QA (Quality Assurance)
- sponsor confirmed

# [Potential overflow at ``updateBaseRate()`` function](https://github.com/code-423n4/2022-06-canto-v2-findings/issues/142) 

# Lines of code

https://github.com/Plex-Engineer/lending-market-v2/blob/443a8c0fed3c5018e95f3881a31b81a555c42b2d/contracts/NoteInterest.sol#L145-L147


# Vulnerability details

## Impact
When casting to ``int`` from ``uint``, the overflow might happen.

## Proof of Concept
https://github.com/Plex-Engineer/lending-market-v2/blob/443a8c0fed3c5018e95f3881a31b81a555c42b2d/contracts/NoteInterest.sol#L145-L147


```
            uint twapMantissa = oracle.getUnderlyingPrice(cNote); // returns price as mantissa
            //uint ir = (1 - twapMantissa).mul(adjusterCoefficient).add(baseRatePerYear);
            int diff = BASE - int(twapMantissa); //possible annoyance if 1e18 - twapMantissa > 2**255, differ
```

``int(twapMantissa)`` can overflow depending on the value of ``uint twapMantissa``. Even if this is not expected, handling this case should be good.

## Tools Used
Static analysis

## Recommended Mitigation Steps
Consider using the logic of ``toInt256`` provided by OpenZeppelin.

https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/utils/math/SafeCast.sol#L1130-L1134

