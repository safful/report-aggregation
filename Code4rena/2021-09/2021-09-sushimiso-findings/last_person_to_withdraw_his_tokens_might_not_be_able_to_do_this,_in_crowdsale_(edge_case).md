## Tags

- bug
- 3 (High Risk)
- sponsor confirmed
- resolved

# [Last person to withdraw his tokens might not be able to do this, in Crowdsale (edge case)](https://github.com/code-423n4/2021-09-sushimiso-findings/issues/15) 

# Handle

gpersoon


# Vulnerability details

## Impact
Suppose a Crowdsale is successful and enough commitments are made before the marketInfo.endTime.
Suppose marketStatus.commitmentsTotal  == marketInfo.totalTokens -1      // note this is an edge case, but can be constructed by an attacker
Then the function auctionEnded() returns true
Assume auctionSuccessful() is also true (might depend on the config of marketPrice.goal and marketInfo.totalTokens)
Then an admin can call finalize() to finalize the Crowdsale.
The function finalize distributes the funds and the unsold tokens and sets status.finalized = true so that finalized cannot be called again.
Now we have "marketInfo.totalTokens -1" tokens left in the contract

However commitEth() or commitTokens() can still be called (they give no error message that the auction has ended)
Then functions call calculateCommitment, which luckily prevent from buying too much, however 1 token can still be bought
These functions also call _addCommitment(), which only checks for marketInfo.endTime, which hasn't passed yet.

Now an extra token is sold and the contract has 1 token short. So the last person to withdraw his tokens cannot withdraw them (because you cannot specify how much you want to withdraw)

Also the revenues for the last token cannot be retrieved as finalize() cannot be called again.

## Proof of Concept
https://github.com/sushiswap/miso/blob/master/contracts/Auctions/Crowdsale.sol#L374
```JS
 function finalize() public nonReentrant {
        require(hasAdminRole(msg.sender) || wallet == msg.sender || hasSmartContractRole(msg.sender) || finalizeTimeExpired(),"Crowdsale: sender must be an admin"); // can be called by admin
        MarketStatus storage status = marketStatus;
        require(!status.finalized, "Crowdsale: already finalized");
        MarketInfo storage info = marketInfo;
        require(auctionEnded(), "Crowdsale: Has not finished yet");    // is true if enough sold, even if this is before marketInfo.endTime

        if (auctionSuccessful()) {          
            /// @dev Transfer contributed tokens to wallet.
            /// @dev Transfer unsold tokens to wallet.
        } else {
            /// @dev Return auction tokens back to wallet.
        }
        status.finalized = true;

function auctionEnded() public view returns (bool) {
        return block.timestamp > uint256(marketInfo.endTime) || 
        _getTokenAmount(uint256(marketStatus.commitmentsTotal) + 1) >= uint256(marketInfo.totalTokens); // is true if enough sold, even if this is before marketInfo.endTime
    }

function auctionSuccessful() public view returns (bool) {
        return uint256(marketStatus.commitmentsTotal) >= uint256(marketPrice.goal);
}

function commitEth(address payable _beneficiary, bool readAndAgreedToMarketParticipationAgreement ) public payable nonReentrant  {
       ...
        uint256 ethToTransfer = calculateCommitment(msg.value);
       ...
       _addCommitment(_beneficiary, ethToTransfer);
   
 function calculateCommitment(uint256 _commitment) public view returns (uint256 committed) { // this prevents buying too much
        uint256 tokens = _getTokenAmount(_commitment);
        uint256 tokensCommited =_getTokenAmount(uint256(marketStatus.commitmentsTotal));
        if ( tokensCommited.add(tokens) > uint256(marketInfo.totalTokens)) {
            return _getTokenPrice(uint256(marketInfo.totalTokens).sub(tokensCommited));
        }
        return _commitment;
    }

function _addCommitment(address _addr, uint256 _commitment) internal {
        require(block.timestamp >= uint256(marketInfo.startTime) && block.timestamp <= uint256(marketInfo.endTime), "Crowdsale: outside auction hours"); // doesn't check auctionEnded() nor status.finalized
        ...
        uint256 newCommitment = commitments[_addr].add(_commitment);
        ...
        commitments[_addr] = newCommitment;

function withdrawTokens(address payable beneficiary) public   nonReentrant  {    
        if (auctionSuccessful()) {
            ...
            uint256 tokensToClaim = tokensClaimable(beneficiary);
            ...
            claimed[beneficiary] = claimed[beneficiary].add(tokensToClaim);
            _safeTokenPayment(auctionToken, beneficiary, tokensToClaim);    // will fail is last token is missing
        } else {



## Tools Used

## Recommended Mitigation Steps
In the function _addCommitment, add a check on auctionEnded() or status.finalized

