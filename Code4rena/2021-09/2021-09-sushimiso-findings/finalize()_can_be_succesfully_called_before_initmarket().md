## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [finalize() can be succesfully called before initMarket()](https://github.com/code-423n4/2021-09-sushimiso-findings/issues/16) 

# Handle

gpersoon


# Vulnerability details

## Impact
The function finalize() of all the auction contracts can be called by everyone before initMarket() is called.
This will set status.finalized = true, which will probably not be detected until the auction is over (because it is only used in a few locations).

If this would happen then the auction cannot be finalized again. Also cancelAuction cannot be called.

Luckily the deployment of the auction contracts is done from createMarket in MISOMarket.sol, which directly calls initMarket().
So in practice this won't pose a problem, however future developers or forks might not be aware of this and deploy the contract differently.

## Proof of Concept
https://github.com/sushiswap/miso/blob/master/contracts/Auctions/Crowdsale.sol#L374

function finalize() public nonReentrant {
        require(hasAdminRole(msg.sender) || wallet == msg.sender || hasSmartContractRole(msg.sender) 
            || finalizeTimeExpired(),                         // initially  true
            "Crowdsale: sender must be an admin"
        );
        MarketStatus storage status = marketStatus;
        require(!status.finalized, "Crowdsale: already finalized");  // initially status.finalized==false
        MarketInfo storage info = marketInfo;
        require(auctionEnded(), "Crowdsale: Has not finished yet");   // initially  true

        if (auctionSuccessful()) {  // initially  true
            /// @dev Successful auction
            /// @dev Transfer contributed tokens to wallet.
            _safeTokenPayment(paymentCurrency, wallet, uint256(status.commitmentsTotal));
            /// @dev Transfer unsold tokens to wallet.
            uint256 soldTokens = _getTokenAmount(uint256(status.commitmentsTotal));
            uint256 unsoldTokens = uint256(info.totalTokens).sub(soldTokens);
            if(unsoldTokens > 0) {
                _safeTokenPayment(auctionToken, wallet, unsoldTokens);
            }
        } else {
            /// @dev Failed auction
            /// @dev Return auction tokens back to wallet.
            _safeTokenPayment(auctionToken, wallet, uint256(info.totalTokens));
        }

        status.finalized = true;    // will end up here

        emit AuctionFinalized();
    }

function finalizeTimeExpired() public view returns (bool) {   
        return uint256(marketInfo.endTime) + 7 days < block.timestamp;  // initially  true (0 + 7 days <  block.timestamp)
    }
    
 function auctionSuccessful() public view returns (bool) {
        return uint256(marketStatus.commitmentsTotal) >= uint256(marketPrice.goal); // initially  true  (0>=0)
    }

 function auctionEnded() public view returns (bool) {
        return block.timestamp > uint256(marketInfo.endTime) ||                              // // initially  true  (block.timestamp>0)
        _getTokenAmount(uint256(marketStatus.commitmentsTotal) + 1) >= uint256(marketInfo.totalTokens);
    }

// https://github.com/sushiswap/miso/blob/master/contracts/MISOMarket.sol#L273
function createMarket(...)     {
        newMarket = deployMarket(_templateId, _integratorFeeAccount);
        ...
        IMisoMarket(newMarket).initMarket(_data);

## Tools Used

## Recommended Mitigation Steps
In function finalize() add something like:
    require(isInitialized(),"Not initialized");


