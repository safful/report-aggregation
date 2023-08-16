## Tags

- bug
- 3 (High Risk)
- sponsor confirmed
- disagree with severity
- resolved

# [Can access cards of other markets](https://github.com/code-423n4/2021-06-realitycards-findings/issues/11) 

# Handle

gpersoon


# Vulnerability details

## Impact
Within RCMarket.sol the functions ownerOf and onlyTokenOwner do not check if the _cardId/_token is smaller than numberOfCards.
So it's possible to supply a larger number and access cards of other markets.
The most problematic seems to be upgradeCard. Here the check for isMarketApproved can be circumvented by trying to move the card via another market.

You can still only move cards you own.

## Proof of Concept
// https://github.com/code-423n4/2021-06-realitycards/blob/main/contracts/RCMarket.sol#L338
    function ownerOf(uint256 _cardId) public view override returns (address) {
        uint256 _tokenId = _cardId + totalNftMintCount; // doesn't check if _cardId < numberOfCards
        return nfthub.ownerOf(_tokenId);
    }

https://github.com/code-423n4/2021-06-realitycards/blob/main/contracts/RCMarket.sol#L313
  modifier onlyTokenOwner(uint256 _token) {
        require(msgSender() == ownerOf(_token), "Not owner"); // _token could be higher than numberOfCards,
        _;
    }

  function upgradeCard(uint256 _card) external onlyTokenOwner(_card) {   // _card  could be higher than numberOfCards,
        _checkState(States.WITHDRAW);
        require(
            !factory.trapIfUnapproved() ||
                factory.isMarketApproved(address(this)),   // this can be circumvented by calling the function via another market
            "Upgrade blocked"
        );
        uint256 _tokenId = _card + totalNftMintCount;    // _card  could be higher than numberOfCards, thus accessing a card in another market
        _transferCard(ownerOf(_card), address(this), _card); // contract becomes final resting place
        nfthub.withdrawWithMetadata(_tokenId);
        emit LogNftUpgraded(_card, _tokenId);
    }
 

## Tools Used

## Recommended Mitigation Steps
Add the following to ownerOf:
require(_card < numberOfCards, "Card does not exist");


