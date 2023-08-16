## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed
- Resolved

# [remove addMarket from RCNftHubL2](https://github.com/code-423n4/2021-08-realitycards-findings/issues/11) 

# Handle

gpersoon


# Vulnerability details

## Impact
The contract RCNftHubL2 contains a function addMarket and an array isMarket.
This was useful in the previous version of the contract, but now it is no longer used.
Note: isMarket() could be used to retrieve the markets from RCNftHubL2, but there are also other ways to do that.

## Proof of Concept
// https://github.com/code-423n4/2021-08-realitycards/blob/main/contracts/nfthubs/RCNftHubL2.sol#L31
/// @dev so only markets can move NFTs
    mapping(address => bool) public isMarket;

 /// @dev so only markets can change ownership
    function addMarket(address _newMarket) external override {
        require(msgSender() == address(factory), "Not factory");
        isMarket[_newMarket] = true;
    }

    // MARKET ONLY
    function transferNft( address _currentOwner,  address _newOwner,   uint256 _tokenId ) external override {
        require(marketTracker[_tokenId] == msgSender(), "Not market");
        _transfer(_currentOwner, _newOwner, _tokenId);
    }

//https://github.com/code-423n4/2021-08-realitycards/blob/main/contracts/RCFactory.sol#L636
function createMarket(
...
 nfthub.addMarket(_newAddress);

## Tools Used

## Recommended Mitigation Steps
Double check if isMarket and addMarket have any use left.
If not remove them from RCNftHubL2
Also remove the call to nfthub.addMarket(_newAddress) from RCFactory.sol

