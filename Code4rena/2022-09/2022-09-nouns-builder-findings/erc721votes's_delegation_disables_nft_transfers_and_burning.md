## Tags

- bug
- 3 (High Risk)
- sponsor confirmed
- old-submission-method

# [ERC721Votes's delegation disables NFT transfers and burning](https://github.com/code-423n4/2022-09-nouns-builder-findings/issues/373) 

# Lines of code

https://github.com/code-423n4/2022-09-nouns-builder/blob/7e9fddbbacdd7d7812e912a369cfd862ee67dc03/src/lib/token/ERC721Votes.sol#L262-L268


# Vulnerability details

If Alice the NFT owner first delegates her votes to herself, second delegates to anyone else with delegate() or delegateBySig() then all her NFT ids will become stuck: their transfers and burning will be disabled.

The issue is _afterTokenTransfer() callback running the _moveDelegateVotes() with an owner instead of her delegate. As Alice's votes in the checkpoint is zero after she delegated them, the subtraction _moveDelegateVotes() tries to perform during the move of the votes will be reverted.

As ERC721Votes is parent to Token and delegate is a kind of common and frequent operation, the impact is governance token moves being frozen in a variety of use cases, which interferes with governance voting process and can be critical for the project.

## Proof of Concept

Suppose Alice delegated all her votes to herself and then decided to delegate them to someone else with either delegate() or delegateBySig() calling _delegate():

https://github.com/code-423n4/2022-09-nouns-builder/blob/7e9fddbbacdd7d7812e912a369cfd862ee67dc03/src/lib/token/ERC721Votes.sol#L179-L190

```solidity
    function _delegate(address _from, address _to) internal {
        // Get the previous delegate
        address prevDelegate = delegation[_from];

        // Store the new delegate
        delegation[_from] = _to;

        emit DelegateChanged(_from, prevDelegate, _to);

        // Transfer voting weight from the previous delegate to the new delegate
        _moveDelegateVotes(prevDelegate, _to, balanceOf(_from));
    }
```

_moveDelegateVotes() will set her votes to `0` as `_from == Alice` and `prevTotalVotes = _amount = balanceOf(Alice)` (as _afterTokenTransfer() incremented Alice's vote balance on each mint to her):

https://github.com/code-423n4/2022-09-nouns-builder/blob/7e9fddbbacdd7d7812e912a369cfd862ee67dc03/src/lib/token/ERC721Votes.sol#L196-L217

```solidity
    function _moveDelegateVotes(
        address _from,
        address _to,
        uint256 _amount
    ) internal {
        unchecked {
            // If voting weight is being transferred:
            if (_from != _to && _amount > 0) {
                // If this isn't a token mint:
                if (_from != address(0)) {
                    // Get the sender's number of checkpoints
                    uint256 nCheckpoints = numCheckpoints[_from]++;

                    // Used to store the sender's previous voting weight
                    uint256 prevTotalVotes;

                    // If this isn't the sender's first checkpoint: Get their previous voting weight
                    if (nCheckpoints != 0) prevTotalVotes = checkpoints[_from][nCheckpoints - 1].votes;

                    // Update their voting weight
                    _writeCheckpoint(_from, nCheckpoints, prevTotalVotes, prevTotalVotes - _amount);
                }
```

After that her votes in the checkpoint become zero. She will not be able to transfer the NFT as `_afterTokenTransfer` will revert on `_moveDelegateVotes`'s attempt to move `1` vote from `Alice` to `_to`, while `checkpoints[Alice][nCheckpoints - 1].votes` is `0`:

https://github.com/code-423n4/2022-09-nouns-builder/blob/7e9fddbbacdd7d7812e912a369cfd862ee67dc03/src/lib/token/ERC721Votes.sol#L262-L268

```solidity
    function _afterTokenTransfer(
        address _from,
        address _to,
        uint256 _tokenId
    ) internal override {
        // Transfer 1 vote from the sender to the recipient
        _moveDelegateVotes(_from, _to, 1);
```

## Recommended Mitigation Steps

The root issue is _afterTokenTransfer() dealing with Alice instead of Alice's delegate.

Consider including delegates() call as a fix:

https://github.com/code-423n4/2022-09-nouns-builder/blob/7e9fddbbacdd7d7812e912a369cfd862ee67dc03/src/lib/token/ERC721Votes.sol#L262-L268

```solidity
    function _afterTokenTransfer(
        address _from,
        address _to,
        uint256 _tokenId
    ) internal override {
        // Transfer 1 vote from the sender to the recipient
-       _moveDelegateVotes(_from, _to, 1);
+       _moveDelegateVotes(delegates(_from), delegates(_to), 1);
```

As `delegates(address(0)) == address(0)` the burning/minting flow will persist:

https://github.com/code-423n4/2022-09-nouns-builder/blob/7e9fddbbacdd7d7812e912a369cfd862ee67dc03/src/lib/token/ERC721Votes.sol#L124-L129

```solidity
    /// @notice The delegate for an account
    /// @param _account The account address
    function delegates(address _account) external view returns (address) {
        address current = delegation[_account];
        return current == address(0) ? _account : current;
    }
```


