## Tags

- bug
- disagree with severity
- QA (Quality Assurance)
- sponsor confirmed

# [Issue with totalSupplyAvg value calculation ](https://github.com/code-423n4/2022-09-canto-findings/issues/134) 

# Lines of code

https://github.com/code-423n4/2022-09-canto/blob/65fbb8b9de22cf8f8f3d742b38b4be41ee35c468/src/Swap/BaseV1-core.sol#L260-L269


# Vulnerability details

## Impact
BaseV1-core.sol#L260
`totalSupplyAvg` may not provide the average value when `granularity` is lesser than or greater(too away from median value) than the total number of `_totalSupplyAvg`

## Proof of Concept
https://github.com/code-423n4/2022-09-canto/blob/65fbb8b9de22cf8f8f3d742b38b4be41ee35c468/src/Swap/BaseV1-core.sol#L260-L269

    function totalSupplyAvg(uint granularity) external view returns(uint) {
        uint[] memory _totalSupplyAvg = sampleSupply(granularity, 1);
        uint totalSupplyCumulativeAvg;


        for (uint i = 0; i < _totalSupplyAvg.length; ++i) {
            totalSupplyCumulativeAvg += _totalSupplyAvg[i]; //totalSupply denominated in terms of 1e18 
        }


        return (totalSupplyCumulativeAvg / granularity);
    }

In above code the average is computed based on `granularity` but thie `granularity` can be a value which is too far away from the median value.
say, it could be too away from `_totalSupplyAvg.length`

## Tools Used
VS code and Manual code review

## Recommended Mitigation Steps
It is suggested to calculate the average value based on `_totalSupplyAvg.length`
`totalSupplyCumulativeAvg / _totalSupplyAvg.length`
