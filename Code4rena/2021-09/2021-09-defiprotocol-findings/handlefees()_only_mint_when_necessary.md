## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [handleFees() only mint when necessary](https://github.com/code-423n4/2021-09-defiprotocol-findings/issues/53) 

# Handle

gpersoon


# Vulnerability details

## Impact
If the functions mintTo and burn of Basket.sol are called twice in the same block then block.timestamp will stay the same and timeDiff ==0.
Then it is not necessary to _mint () tokens, as this will be 0 tokens anyway.

So checking for timeDiff ==0 could save a bit of gas.

## Proof of Concept
https://github.com/code-423n4/2021-09-defiProtocol/blob/main/contracts/contracts/Basket.sol#L110
 function handleFees() private {
        if (lastFee == 0) {
            lastFee = block.timestamp;
        } else {
            uint256 startSupply = totalSupply();

            uint256 timeDiff = (block.timestamp - lastFee);
            uint256 feePct = timeDiff * licenseFee / ONE_YEAR;
            uint256 fee = startSupply * feePct / (BASE - feePct);

            _mint(publisher, fee * (BASE - factory.ownerSplit()) / BASE);
            _mint(Ownable(address(factory)).owner(), fee * factory.ownerSplit() / BASE);
            lastFee = block.timestamp;
            uint256 newIbRatio = ibRatio * startSupply / totalSupply();
            ibRatio = newIbRatio;

            emit NewIBRatio(ibRatio);
        }
    }
## Tools Used

## Recommended Mitigation Steps
Add an extra if in the following way:

 function handleFees() private {
        if (lastFee == 0) {
            lastFee = block.timestamp;
        } else {
            uint256 startSupply = totalSupply();
            uint256 timeDiff = (block.timestamp - lastFee);
            if (timeDiff !=0) {                                                                      // ===> extra if
                uint256 feePct = timeDiff * licenseFee / ONE_YEAR;
                uint256 fee = startSupply * feePct / (BASE - feePct);
                _mint(publisher, fee * (BASE - factory.ownerSplit()) / BASE);
                _mint(Ownable(address(factory)).owner(), fee * factory.ownerSplit() / BASE);
                lastFee = block.timestamp;
                uint256 newIbRatio = ibRatio * startSupply / totalSupply();
                ibRatio = newIbRatio;
                emit NewIBRatio(ibRatio);
           }
        }
    }




