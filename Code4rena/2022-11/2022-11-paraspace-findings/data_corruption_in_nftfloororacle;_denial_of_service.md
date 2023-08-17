## Tags

- bug
- 3 (High Risk)
- primary issue
- satisfactory
- selected for report
- sponsor confirmed
- H-01

# [Data corruption in NFTFloorOracle; Denial of Service](https://github.com/code-423n4/2022-11-paraspace-findings/issues/79) 

# Lines of code

https://github.com/code-423n4/2022-11-paraspace/blob/main/paraspace-core/contracts/misc/NFTFloorOracle.sol#L335


# Vulnerability details

## Impact
During `_removeFeeder` operation in `NFTFloorOracle` contract, the feeder is removed from `feeders` array, and linking in `feederPositionMap` for the specific feeder is removed. Deletion logic is implemented in "Swap + Pop" way, so indexes changes, but existing **code doesn't update indexes in** `feederPositionMap` **after feeder removal**, which causes the issue of Denial of Service for further removals.
As a result:
- Impossible to remove some `feeders` from the contract due to Out of Bounds array access. Removal fails because of transaction revert.
- Data in `feederPositionMap` is corrupted after some `feeders` removal. Data linking from `feederPositionMap.index` to `feeders` array is broken. 

## Proof of Concept
```
    address internal feederA = 0x5B38Da6a701c568545dCfcB03FcB875f56beddC4;
    address internal feederB = 0xAb8483F64d9C6d1EcF9b849Ae677dD3315835cb2;
    address internal feederC = 0x4B20993Bc481177ec7E8f571ceCaE8A9e22C02db;

    function corruptFeedersMapping() external {
        console.log("Starting from empty feeders array. Array size: %s", feeders.length);
        address[] memory initialFeeders = new address[](3);
        initialFeeders[0] = feederA;
        initialFeeders[1] = feederB;
        initialFeeders[2] = feederC;
        this.addFeeders(initialFeeders);
        console.log("Feeders array: [%s, %s, %s]", initialFeeders[0], initialFeeders[1], initialFeeders[2]);
        console.log("Remove feeder B");
        this.removeFeeder(feederB);
        console.log("feederPositionMap[A] = %s, feederPositionMap[C] = %s", feederPositionMap[feederA].index, feederPositionMap[feederC].index);
        console.log("Mapping for Feeder C store index 2, which was not updated after removal of B. Feeders array length is : %s", feeders.length);
        console.log("Try remove Feeder C. Transaction will be reverted because of access out of bounds of array. Data is corrupted");
        this.removeFeeder(feederC);
    }
```
Snippet execution result:
![Alt text](https://i.gyazo.com/90ac873cd71194527d4d3b9bfe6e317e.png "Optional title")

## Tools Used
Visual inspection; Solidity snippet for PoC

## Recommended Mitigation Steps
Update index in `feederPositionMap` after feeders swap and pop.
```
feeders[feederIndex] = feeders[feeders.length - 1];
feederPositionMap[feeders[feederIndex]].index = feederIndex; //Index update added as a recommendation
feeders.pop();
```