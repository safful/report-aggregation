## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [set lastFee in initialize() function](https://github.com/code-423n4/2021-09-defiprotocol-findings/issues/94) 

# Handle

itsmeSTYJ


# Vulnerability details

## Impact

Gas optimisation

## Recommended Mitigation Steps

The if branch in the handleFee() function is only there to handle the very first time handleFees are called. Thereafter, this condition will always fail so it makes more sense to initialize it with the initialize() function.

```jsx
function initialize(IFactory.Proposal memory proposal, IAuction auction_) public override {
    publisher = proposal.proposer;
    licenseFee = proposal.licenseFee;
    factory = IFactory(msg.sender);
    auction = auction_;
    ibRatio = BASE;
    tokens = proposal.tokens;
    weights = proposal.weights;
		lastFee = block.timestamp;      // updated lastFee here
    approveUnderlying(address(auction));

    __ERC20_init(proposal.tokenName, proposal.tokenSymbol);
}
...

function handleFees() private {
    // if (lastFee == 0) {            // delete this
    //     lastFee = block.timestamp; // delete this
    // } else {                       // delete this
    uint256 startSupply = totalSupply();

    uint256 timeDiff = (block.timestamp - lastFee);
    uint256 feePct = timeDiff * licenseFee / ONE_YEAR;
    uint256 fee = startSupply * feePct / (BASE - feePct);

    _mint(publisher, fee * (BASE - factory.ownerSplit()) / BASE);
    _mint(Ownable(address(factory)).owner(), fee * factory.ownerSplit() / BASE);
    lastFee = block.timestamp;

    uint256 newIbRatio = ibRatio * startSupply / totalSupply();
    ibRatio = newIbRatio;

    emit NewIBRatio(ibRatio);
    // }                              // delete this
}
```

