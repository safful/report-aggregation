## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [`Token:mint`: infinite loop if the founders' shares sum up to 100](https://github.com/code-423n4/2022-09-nouns-builder-findings/issues/347) 

# Lines of code

https://github.com/code-423n4/2022-09-nouns-builder/blob/7e9fddbbacdd7d7812e912a369cfd862ee67dc03/src/token/Token.sol#L179


# Vulnerability details

## Impact

The Token as well as Auction cannot be used if the sum of `ownershipPct` is 100

## Proof of Concept

```solidity
    function test_poc_mintforever() public {
        createUsers(2, 1 ether);

        address[] memory wallets = new address[](2);
        uint256[] memory percents = new uint256[](2);
        uint256[] memory vestExpirys = new uint256[](2);

        uint256 pct = 50;
        uint256 end = 4 weeks;

        unchecked {
            for (uint256 i; i < 2; ++i) {
                wallets[i] = otherUsers[i];
                percents[i] = pct;
                vestExpirys[i] = end;
            }
        }

        deployWithCustomFounders(wallets, percents, vestExpirys);

        assertEq(token.totalFounders(), 2);
        assertEq(token.totalFounderOwnership(), 100);

        Founder memory founder;

        unchecked {
            for (uint256 i; i < 100; ++i) {
                founder = token.getScheduledRecipient(i);

                if (i % 2 == 0) assertEq(founder.wallet, otherUsers[0]);
                else assertEq(founder.wallet, otherUsers[1]);
            }
        }

// // commented out as it will not stop
//         vm.prank(otherUsers[0]);
//         auction.unpause();

    }
```

In the proof of concept, there are two founders and they both share 50% of ownership. If the `Auction` should be `unpause`d, and therefore triggers to mint tokens, it will go into the infinite loop and eventually revert for out of gas.

```solidity
// Token.sol

143     function mint() external nonReentrant returns (uint256 tokenId) {
144         // Cache the auction address
145         address minter = settings.auction;
146
147         // Ensure the caller is the auction
148         if (msg.sender != minter) revert ONLY_AUCTION();
149
150         // Cannot realistically overflow
151         unchecked {
152             do {
153                 // Get the next token to mint
154                 tokenId = settings.totalSupply++;
155
156                 // Lookup whether the token is for a founder, and mint accordingly if so
157             } while (_isForFounder(tokenId));
158         }
159
160         // Mint the next available token to the auction house for bidding
161         _mint(minter, tokenId);
162     }

177     function _isForFounder(uint256 _tokenId) private returns (bool) {
178         // Get the base token id
179         uint256 baseTokenId = _tokenId % 100;
180
181         // If there is no scheduled recipient:
182         if (tokenRecipient[baseTokenId].wallet == address(0)) {
183             return false;
184
185             // Else if the founder is still vesting:
186         } else if (block.timestamp < tokenRecipient[baseTokenId].vestExpiry) {
187             // Mint the token to the founder
188             _mint(tokenRecipient[baseTokenId].wallet, _tokenId);
189
190             return true;
191
192             // Else the founder has finished vesting:
193         } else {
194             // Remove them from future lookups
195             delete tokenRecipient[baseTokenId];
196
197             return false;
198         }
199     }
```

In the `Token::mint`, there is a while loop which will keep looping as long as `_isForFounder` returns true. The `_isForFounder` function will return true is the given `_tokenId`'s recipient is still vesting. However, to check the recipient it is checking the `baseTokenId` which is `_tokenId % 100` (in line 179 above snippet). Which means, if the `tokenRecipient` of 0 to 99 are currently vesting, it will keep returning true and the while loop in the `mint` function will not stop. The `tokenRecipient` was set in the `_addFounders` and if the sum of all founders' ownership percent is 100, the `tokenRecipient` will be filled up to 100.


## Tools Used

None

## Recommended Mitigation Steps

use `_tokenId` instead of `baseTokenId`.

<!-- zzzitron H01 -->

