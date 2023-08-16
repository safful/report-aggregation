## Tags

- bug
- duplicate
- 1 (Low Risk)
- sponsor confirmed

# [initialize of Basket.sol is missing initializer ](https://github.com/code-423n4/2021-09-defiprotocol-findings/issues/50) 

# Handle

gpersoon


# Vulnerability details

## Impact
When using the Openzeppelin upgradability pattern, the initialize() function should you the modifier initializer.
However the initialize() function of Basket.sol doesn't have this modifier.

This won't give problems in practice because __ERC20_init() does have this modifier and prevents initialize() from being called twice.
However forks of the projects or future developers might not be aware of this any make risky changes.

## Proof of Concept
https://github.com/code-423n4/2021-09-defiProtocol/blob/main/contracts/contracts/Basket.sol#L36
import { ERC20Upgradeable } from "@openzeppelin/contracts-upgradeable/token/ERC20/ERC20Upgradeable.sol";

contract Basket is IBasket, ERC20Upgradeable {
..
 function initialize(IFactory.Proposal memory proposal, IAuction auction_) public override {
       ...

        __ERC20_init(proposal.tokenName, proposal.tokenSymbol);
    }

## Tools Used

## Recommended Mitigation Steps
Add the modifier initializer to the function initialize()

