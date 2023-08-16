## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Minimize Storage Slots (Auction.sol)](https://github.com/code-423n4/2021-10-defiprotocol-findings/issues/46) 

# Handle

ye0lde


# Vulnerability details

## Impact
It is possible to minimize the number of storage slots used by rearranging the state variables in a more efficient way.

## Proof of Concept
In Auction.sol:
https://github.com/code-423n4/2021-10-defiprotocol/blob/7ca848f2779e2e64ed0b4756c02f0137ecd73e50/contracts/contracts/Auction.sol#L16-L28

## Tools Used
Visual Studio Code, Remix

## Recommended Mitigation Steps
Arrange the bool and address variables such that they fit into the same slot.
For example:
<code>
    uint256 private constant BASE = 1e18;
    uint256 private constant ONE_DAY = 4 * 60 * 24; // one day in blocks
    
    bool public override auctionOngoing;
    bool public override hasBonded;
    bool public override initialized;
    address public override auctionBonder;
    
    uint256 public override auctionStart;
</code>
    


