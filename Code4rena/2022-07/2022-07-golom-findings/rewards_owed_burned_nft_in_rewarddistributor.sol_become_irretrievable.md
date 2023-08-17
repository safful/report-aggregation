## Tags

- bug
- 2 (Med Risk)
- disagree with severity
- sponsor confirmed
- edited-by-warden
- selected-for-report

# [Rewards owed burned NFT in RewardDistributor.sol become irretrievable](https://github.com/code-423n4/2022-07-golom-findings/issues/86) 

# Lines of code

https://github.com/code-423n4/2022-07-golom/blob/e5efa8f9d6dda92a90b8b2c4902320acf0c26816/contracts/rewards/RewardDistributor.sol#L172-L210


# Vulnerability details

## Impact
Rewards owed burned NFT are permanently locked

## Proof of Concept
    function _burn(uint256 _tokenId) internal {
        require(_isApprovedOrOwner(msg.sender, _tokenId), 'caller is not owner nor approved');

        address owner = ownerOf(_tokenId);

        // Clear approval
        approve(address(0), _tokenId);
        // Remove token
        _removeTokenFrom(msg.sender, _tokenId);
        emit Transfer(owner, address(0), _tokenId);
    }

    function _removeTokenFrom(address _from, uint256 _tokenId) internal {
        // Throws if `_from` is not the current owner
        assert(idToOwner[_tokenId] == _from);
        // Change the owner
        idToOwner[_tokenId] = address(0);
        // Update owner token index tracking
        _removeTokenFromOwnerList(_from, _tokenId);
        // Change count tracking
        ownerToNFTokenCount[_from] -= 1;
    }

After an NFT is burned, owner of token is set to address(0).

        rewardToken.transfer(tokenowner, reward);

This causes issues in multiStakerClaim L208. GOLOM uses OZ's implementation of ERC20 which doesn't allow tokens to be sent to address(0). Because the "owner" of the burned NFT is address(0) multiStakerClaim will always revert when called for a burned NFT trapping rewards in contract forever.

## Tools Used

## Recommended Mitigation Steps
Implement a clawback clause inside the multiStakerClaim function. If the token is burned (i.e. owned by address(0)) the rewards should be transferred to different address. These rewards could be claimed to the treasury or burned, etc. 

        if (tokenowner == address(0){
            rewardToken.transfer(treasury, reward);
            weth.transfer(treasury, rewardEth);
        }

