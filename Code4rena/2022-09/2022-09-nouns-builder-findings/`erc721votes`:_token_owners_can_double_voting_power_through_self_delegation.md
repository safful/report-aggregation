## Tags

- bug
- 3 (High Risk)
- sponsor confirmed
- edited-by-warden

# [`ERC721Votes`: Token owners can double voting power through self delegation](https://github.com/code-423n4/2022-09-nouns-builder-findings/issues/413) 

# Lines of code

https://github.com/code-423n4/2022-09-nouns-builder/blob/7e9fddbbacdd7d7812e912a369cfd862ee67dc03/src/lib/token/ERC721Votes.sol#L131-L135
https://github.com/code-423n4/2022-09-nouns-builder/blob/7e9fddbbacdd7d7812e912a369cfd862ee67dc03/src/lib/token/ERC721Votes.sol#L176-L190
https://github.com/code-423n4/2022-09-nouns-builder/blob/7e9fddbbacdd7d7812e912a369cfd862ee67dc03/src/lib/token/ERC721Votes.sol#L192-L235


# Vulnerability details

The owner of one or many `ERC721Votes` tokens can double their voting power once (and only once) by delegating to their own address as their first delegation.

### Scenario
This exploit relies on the initial default value of the `delegation` mapping in `ERC721Votes`, which is why it will only work once per address.

First, the token owner must call `delegate` or `delegateBySig`, passing their own address as the delegate:

[`ERC721Votes#delegate`](https://github.com/code-423n4/2022-09-nouns-builder/blob/7e9fddbbacdd7d7812e912a369cfd862ee67dc03/src/lib/token/ERC721Votes.sol#L131-L135)

```solidity
    /// @notice Delegates votes to an account
    /// @param _to The address delegating votes to
    function delegate(address _to) external {
        _delegate(msg.sender, _to);
    }
```

This calls into the internal `_delegate` function, with `_from` and `_to` both set to the token owner's address:

[`ERC721Votes#_delegate`](https://github.com/code-423n4/2022-09-nouns-builder/blob/7e9fddbbacdd7d7812e912a369cfd862ee67dc03/src/lib/token/ERC721Votes.sol#L176-L190)

```solidity
    /// @dev Updates delegate addresses
    /// @param _from The address delegating votes from
    /// @param _to The address delegating votes to
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

Since this is the token owner's first delegation, the `delegation` mapping does not contain a value for the `_from` address, and `prevDelegate` on L#181 will be set to `address(0)`:

[`ERC721Votes.sol#L180-L181`](https://github.com/code-423n4/2022-09-nouns-builder/blob/7e9fddbbacdd7d7812e912a369cfd862ee67dc03/src/lib/token/ERC721Votes.sol#L180-L181)

```solidity
        // Get the previous delegate
        address prevDelegate = delegation[_from];
```

This function then calls into `_moveDelegateVotes` to transfer voting power. This time, `_from` is `prevDelegate`, equal to `address(0)`; `_to` is the token owner's address; and `_amount` is `balanceOf(_from)`, the token owner's current balance:

[`ERC721Votes#_moveDelegateVotes`](https://github.com/code-423n4/2022-09-nouns-builder/blob/7e9fddbbacdd7d7812e912a369cfd862ee67dc03/src/lib/token/ERC721Votes.sol#L192-L235)

```solidity
 /// @dev Transfers voting weight
    /// @param _from The address delegating votes from
    /// @param _to The address delegating votes to
    /// @param _amount The number of votes delegating
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

                // If this isn't a token burn:
                if (_to != address(0)) {
                    // Get the recipients's number of checkpoints
                    uint256 nCheckpoints = numCheckpoints[_to]++;

                    // Used to store the recipient's previous voting weight
                    uint256 prevTotalVotes;

                    // If this isn't the recipient's first checkpoint: Get their previous voting weight
                    if (nCheckpoints != 0) prevTotalVotes = checkpoints[_to][nCheckpoints - 1].votes;

                    // Update their voting weight
                    _writeCheckpoint(_to, nCheckpoints, prevTotalVotes, prevTotalVotes + _amount);
                }
            }
        }
    }
```

The `if` condition on L#203 is `true`, since `_from` is `address(0)`, `_to` is the owner address, and `_amount` is nonzero:

[`ERC721Votes.sol#L202-L203`](https://github.com/code-423n4/2022-09-nouns-builder/blob/7e9fddbbacdd7d7812e912a369cfd862ee67dc03/src/lib/token/ERC721Votes.sol#L202-L203)

```solidity
            // If voting weight is being transferred:
            if (_from != _to && _amount > 0) {
```

Execution skips the `if` block on L#205-217, since `_from` is `address(0)`:

[`ERC721Votes.sol#L205-L217`](https://github.com/code-423n4/2022-09-nouns-builder/blob/7e9fddbbacdd7d7812e912a369cfd862ee67dc03/src/lib/token/ERC721Votes.sol#L204-L217)

```solidity
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

However, the `if` block on L#220-232 will execute and increase the voting power allocated to `_to`:

[`ERC721Votes.sol#L220-L232`](https://github.com/code-423n4/2022-09-nouns-builder/blob/7e9fddbbacdd7d7812e912a369cfd862ee67dc03/src/lib/token/ERC721Votes.sol#L219-L232)

```solidity
                // If this isn't a token burn:
                if (_to != address(0)) {
                    // Get the recipients's number of checkpoints
                    uint256 nCheckpoints = numCheckpoints[_to]++;

                    // Used to store the recipient's previous voting weight
                    uint256 prevTotalVotes;

                    // If this isn't the recipient's first checkpoint: Get their previous voting weight
                    if (nCheckpoints != 0) prevTotalVotes = checkpoints[_to][nCheckpoints - 1].votes;

                    // Update their voting weight
                    _writeCheckpoint(_to, nCheckpoints, prevTotalVotes, prevTotalVotes + _amount);
                }
```

The token owner's voting power has now been increased by an amount equal to their total number of tokens, without an offsetting decrease.

This exploit only works once: if a token owner subsequently delegates to themselves after their initial self delegation, `prevDelegate` will be set to a non-default value in `_delegate`, and the delegation logic will work as intended.

### Impact
Malicious `ERC21Votes` owners can accrue more voting power than they deserve. Especially malicious owners may quietly acquire multiple tokens before doubling their voting power. In an early DAO with a small supply of tokens, the impact of this exploit could be significant.

### Recommendation
Make the `delegates` function `public` rather than `external`:

```solidity
    /// @notice The delegate for an account
    /// @param _account The account address
    function delegates(address _account) public view returns (address) {
        address current = delegation[_account];
        return current == address(0) ? _account : current;
    }
```

Then, call this function rather than accessing the `delegation` mapping directly:

```solidity
    /// @dev Updates delegate addresses
    /// @param _from The address delegating votes from
    /// @param _to The address delegating votes to
    function _delegate(address _from, address _to) internal {
        // Get the previous delegate
        address prevDelegate = delegates(_from);

        // Store the new delegate
        delegation[_from] = _to;

        emit DelegateChanged(_from, prevDelegate, _to);

        // Transfer voting weight from the previous delegate to the new delegate
        _moveDelegateVotes(prevDelegate, _to, balanceOf(_from));
    }
```

Note that the original NounsDAO contracts follow this pattern. (See [here](https://github.com/nounsDAO/nouns-monorepo/blob/master/packages/nouns-contracts/contracts/base/ERC721Checkpointable.sol#L83-L91) and [here](https://github.com/nounsDAO/nouns-monorepo/blob/master/packages/nouns-contracts/contracts/base/ERC721Checkpointable.sol#L83-L91)).


### Test cases
(Put the following test cases in `Gov.t.sol`)

```solidity
    function test_delegate_to_self_doubles_voting_power() public {
        mintVoter1();

        assertEq(token.getVotes(address(voter1)), 1);

        vm.startPrank(voter1);
        token.delegate(address(voter1));

        assertEq(token.getVotes(address(voter1)), 2);
    }

    function mintToken(uint256 tokenId) internal {
        vm.prank(voter1);
        auction.createBid{ value: 0.420 ether }(tokenId);

        vm.warp(block.timestamp + auctionParams.duration + 1 seconds);
        auction.settleCurrentAndCreateNewAuction();
    }

    function test_delegate_to_self_multiple_tokens_doubles_voting_power() public {
        // An especially malicious user may acquire multiple tokens
        // before doubling their voting power through this exploit.
        mintVoter1();
        mintToken(3);
        mintToken(4);
        mintToken(5);
        mintToken(6);

        assertEq(token.getVotes(address(voter1)), 5);

        vm.prank(voter1);
        token.delegate(address(voter1));

        assertEq(token.getVotes(address(voter1)), 10);
    }
```