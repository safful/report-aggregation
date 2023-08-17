## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed
- selected for report
- responded

# [PA1D#bidSharesForToken returns incorrect bidShares.creator.value](https://github.com/code-423n4/2022-10-holograph-findings/issues/180) 

# Lines of code

https://github.com/code-423n4/2022-10-holograph/blob/f8c2eae866280a1acfdc8a8352401ed031be1373/contracts/enforcer/PA1D.sol#L665-L675


# Vulnerability details

## Impact

bidShares returned are incorrect leading to incorrect royalties

## Proof of Concept

[Zora Market](https://etherscan.io/address/0xe5bfab544eca83849c53464f85b7164375bdaac1#code#F1#L113)

    function isValidBidShares(BidShares memory bidShares)
        public
        pure
        override
        returns (bool)
    {
        return
            bidShares.creator.value.add(bidShares.owner.value).add(
                bidShares.prevOwner.value
            ) == uint256(100).mul(Decimal.BASE);
    }

Above you can see the Zora market lines that validate bidShares, which shows that Zora market bidShare.values should be percentages written out to 18 decimals. However PA1D#bidSharesForToken sets the bidShares.creator.value to the raw basis points set by the owner, which is many order of magnitudes different than expected.   

## Tools Used

Manual Review

## Recommended Mitigation Steps

To return the proper value, basis points returned need to be adjusted. Convert from basis points to percentage by dividing by 10 ** 2 (100) then scale to 18 decimals. The final result it to multiple the basis point by 10 ** (18 - 2) or 10 ** 16: 

    function bidSharesForToken(uint256 tokenId) public view returns (ZoraBidShares memory bidShares) {
        // this information is outside of the scope of our
        bidShares.prevOwner.value = 0;
        bidShares.owner.value = 0;
        if (_getReceiver(tokenId) == address(0)) {
    -       bidShares.creator.value = _getDefaultBp();
    +       bidShares.creator.value = _getDefaultBp() * (10 ** 16);
        } else {
    -       bidShares.creator.value = _getBp(tokenId);
    +       bidShares.creator.value = _getBp(tokenId) * (10 ** 16);
        }
        return bidShares;
    }