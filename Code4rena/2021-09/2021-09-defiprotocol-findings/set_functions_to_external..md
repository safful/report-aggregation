## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Set functions to external.](https://github.com/code-423n4/2021-09-defiprotocol-findings/issues/240) 

# Handle

goatbug


# Vulnerability details

## Impact
Long list of functions should be set from public to external since they are not called anywhere by the contract itself. 



## Proof of Concept

There are too many to list them all from all the contracts.

Just some examples in the Factory contract.

There are lots in every contract that should rather be external.

    function setMinLicenseFee(uint256 newMinLicenseFee) public override onlyOwner {
        minLicenseFee = newMinLicenseFee;
    }

    function setAuctionDecrement(uint256 newAuctionDecrement) public override onlyOwner {
        auctionDecrement = newAuctionDecrement;
    }

    function setAuctionMultiplier(uint256 newAuctionMultiplier) public override onlyOwner {
        auctionMultiplier = newAuctionMultiplier;
    }

    function setBondPercentDiv(uint256 newBondPercentDiv) public override onlyOwner {
        bondPercentDiv = newBondPercentDiv;
    }

    function setOwnerSplit(uint256 newOwnerSplit) public override onlyOwner {
        require(newOwnerSplit <= 2e17); // 20%

        ownerSplit = newOwnerSplit;
    }

## Tools Used

## Recommended Mitigation Steps

