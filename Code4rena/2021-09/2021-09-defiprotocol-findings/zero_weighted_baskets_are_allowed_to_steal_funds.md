## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Zero weighted baskets are allowed to steal funds](https://github.com/code-423n4/2021-09-defiprotocol-findings/issues/21) 

# Handle

csanuragjain


# Vulnerability details

## Impact
It was observed that Publisher is allowed to create a basket with zero token and weight. This can lead to user fund stealing as described in below poc
The issue was discovered in validateWeights function of Basket contract 

## Proof of Concept
1. User proposes a new Basket with 0 tokens and weights using proposeBasketLicense function in Factory contract

```
 Proposal memory proposal = Proposal({
            licenseFee: 10,
            tokenName: abc,
            tokenSymbol: aa,
            proposer: 0xabc,
            tokens: {},
            weights: {},
            basket: address(0)
        });
```

2. validateWeights function is called and it returns success as the only check performed is _tokens.length == _weights.length (0=0)

```
    function validateWeights(address[] memory _tokens, uint256[] memory _weights) public override pure {
        require(_tokens.length == _weights.length);
        uint256 length = _tokens.length;
        address[] memory tokenList = new address[](length);

        // check uniqueness of tokens and not token(0)

        for (uint i = 0; i < length; i++) {
           ...
        }
    }
```

3. A new proposal gets created

```
_proposals.push(proposal);
```

4. User creates new Basket with this proposal using createBasket function

```
function createBasket(uint256 idNumber) external override returns (IBasket) {
        Proposal memory bProposal = _proposals[idNumber];
        require(bProposal.basket == address(0));

        ....

        for (uint256 i = 0; i < bProposal.weights.length; i++) {
            ...
        }
		...
        return newBasket;
    }
```

5. Since no weights and tokens were in this proposal so no token transfer is required (bProposal.weights.length will be 0 so loop won't run)

6. Basket gets created and user becomes publisher for this basket

```
        newBasket.mintTo(BASE, msg.sender);
        _proposals[idNumber].basket = address(newBasket);
```

7. Publisher owned address calls the mint function with say amount 10 on Basket.sol contract

```
    function mint(uint256 amount) public override {
        mintTo(amount, msg.sender);
    }

    function mintTo(uint256 amount, address to) public override {
        ...

        pullUnderlying(amount, msg.sender);

        _mint(to, amount);

        ...
    }
```

8. Since there is no weights so pullUnderlying function does nothing (weights.length is 0)

```
    function pullUnderlying(uint256 amount, address from) private {
        for (uint256 i = 0; i < weights.length; i++) {
            uint256 tokenAmount = amount * weights[i] * ibRatio / BASE / BASE;
            IERC20(tokens[i]).safeTransferFrom(from, address(this), tokenAmount);
        }
    }
```

9. Full amount 10 is minted to Publisher owned address setting balanceOf(msg.sender) = 10

```
_mint(to, amount);
```

10. Now Publisher calls the publishNewIndex to set new weights. Since pendingWeights.pending is false, else condition gets executed

```
    function publishNewIndex(address[] memory _tokens, uint256[] memory _weights) onlyPublisher public override {
        validateWeights(_tokens, _weights);

        if (pendingWeights.pending) {
            require(block.number >= pendingWeights.block + TIMELOCK_DURATION);
            if (auction.auctionOngoing() == false) {
                auction.startAuction();

                emit PublishedNewIndex(publisher);
            } else if (auction.hasBonded()) {

            } else {
                auction.killAuction();

                pendingWeights.tokens = _tokens;
                pendingWeights.weights = _weights;
                pendingWeights.block = block.number;
            }
        } else {
            pendingWeights.pending = true;
            pendingWeights.tokens = _tokens;
            pendingWeights.weights = _weights;
            pendingWeights.block = block.number;
        }
    }
```
 
11. Publisher calls the publishNewIndex again which starts the Auction. This auction is later settled using the settleAuction function in Auction contract

12. Publisher owned address can now call burn and get the amount 10 even though he never made the payment since his balanceOf(msg.sender) = 10 (Step 9)

```
    function burn(uint256 amount) public override {
        require(auction.auctionOngoing() == false);
        require(amount > 0);
        require(balanceOf(msg.sender) >= amount);

        handleFees();

        pushUnderlying(amount, msg.sender);
        _burn(msg.sender, amount);
        
        emit Burned(msg.sender, amount);
    }
```

## Recommended Mitigation Steps
Change validateWeights to check for 0 length token

```
    function validateWeights(address[] memory _tokens, uint256[] memory _weights) public override pure {
        require(_tokens.length>0);
		...
    }
```

