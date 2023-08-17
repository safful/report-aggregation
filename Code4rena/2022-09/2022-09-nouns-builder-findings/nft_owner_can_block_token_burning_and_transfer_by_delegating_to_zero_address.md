## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed
- old-submission-method

# [NFT owner can block token burning and transfer by delegating to zero address](https://github.com/code-423n4/2022-09-nouns-builder-findings/issues/478) 

# Lines of code

https://github.com/code-423n4/2022-09-nouns-builder/blob/7e9fddbbacdd7d7812e912a369cfd862ee67dc03/src/lib/token/ERC721Votes.sol#L179-L190


# Vulnerability details

ERC721Votes's delegate() and delegateBySig() allow the delegation to zero address, which result in owner's votes elimination in the checkpoint. I.e. the votes are subtracted from the owner, but aren't added anywhere. _moveDelegateVotes() invoked by _delegate() treats the corresponding call as a burning, erasing the votes.

The impact is that the further transfer and burning attempts for the ids of the owner will be reverted because _afterTokenTransfer() callback will try to reduce owner's votes, which are already zero, reverting the calls due to subtraction fail.

As ERC721Votes is parent to Token the overall impact is governance token burning and transfer being disabled whenever the owner delegated to zero address. This can be done deliberately, i.e. any owner can disable burning and transfer of the owned ids at any moment, which can interfere with governance voting process.

## Proof of Concept

User facing delegate() and delegateBySig() allow for zero address delegation:

https://github.com/code-423n4/2022-09-nouns-builder/blob/7e9fddbbacdd7d7812e912a369cfd862ee67dc03/src/lib/token/ERC721Votes.sol#L131-L135

```solidity
    /// @notice Delegates votes to an account
    /// @param _to The address delegating votes to
    function delegate(address _to) external {
        _delegate(msg.sender, _to);
    }
```

https://github.com/code-423n4/2022-09-nouns-builder/blob/7e9fddbbacdd7d7812e912a369cfd862ee67dc03/src/lib/token/ERC721Votes.sol#L144-L174

```solidity
    function delegateBySig(
        address _from,
        address _to,
        uint256 _deadline,
        uint8 _v,
        bytes32 _r,
        bytes32 _s
    ) external {
        // Ensure the signature has not expired
        if (block.timestamp > _deadline) revert EXPIRED_SIGNATURE();

        // Used to store the digest
        bytes32 digest;

        // Cannot realistically overflow
        unchecked {
            // Compute the hash of the domain seperator with the typed delegation data
            digest = keccak256(
                abi.encodePacked("\x19\x01", DOMAIN_SEPARATOR(), keccak256(abi.encode(DELEGATION_TYPEHASH, _from, _to, nonces[_from]++, _deadline)))
            );
        }

        // Recover the message signer
        address recoveredAddress = ecrecover(digest, _v, _r, _s);

        // Ensure the recovered signer is the voter
        if (recoveredAddress == address(0) || recoveredAddress != _from) revert INVALID_SIGNATURE();

        // Update the delegate
        _delegate(_from, _to);
    }
```

And pass zero address to the _delegate() where it is being set:

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

In this case _moveDelegateVotes() will reduce the votes from the owner, not adding it to anywhere as `_from` is the owner, while `_to` is zero address:

https://github.com/code-423n4/2022-09-nouns-builder/blob/7e9fddbbacdd7d7812e912a369cfd862ee67dc03/src/lib/token/ERC721Votes.sol#L203-L220

```solidity
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

                // If this isn't a token burn:
                if (_to != address(0)) { // @ audit here we add the votes to the target, but only if it's not zero address
```

The owner might know that and can use such a delegation to interfere with the system by prohibiting of transferring/burning of his ids.

This happens via _afterTokenTransfer() reverting as it's becomes impossible to reduce owner's votes balance by `1`:

https://github.com/code-423n4/2022-09-nouns-builder/blob/7e9fddbbacdd7d7812e912a369cfd862ee67dc03/src/lib/token/ERC721Votes.sol#L262-L271

```solidity
    function _afterTokenTransfer(
        address _from,
        address _to,
        uint256 _tokenId
    ) internal override {
        // Transfer 1 vote from the sender to the recipient
        _moveDelegateVotes(_from, _to, 1);

        super._afterTokenTransfer(_from, _to, _tokenId);
    }
```

## Recommended Mitigation Steps

Consider prohibiting zero address as a delegation destination:

https://github.com/code-423n4/2022-09-nouns-builder/blob/7e9fddbbacdd7d7812e912a369cfd862ee67dc03/src/lib/token/ERC721Votes.sol#L179-L190

```solidity
    function _delegate(address _from, address _to) internal {
+		if (_to == address(0)) revert INVALID_SIGNATURE();

        // Get the previous delegate
        address prevDelegate = delegation[_from];

        // Store the new delegate
        delegation[_from] = _to;

        emit DelegateChanged(_from, prevDelegate, _to);

        // Transfer voting weight from the previous delegate to the new delegate
        _moveDelegateVotes(prevDelegate, _to, balanceOf(_from));
    }
```

When `_to` isn't zero there always be an addition in _moveDelegateVotes(), so the system votes balance will be sustained.



