## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [sweepToken() function removed in CErc20.sol from original Compound code](https://github.com/code-423n4/2021-04-basedloans-findings/issues/17) 

# Handle

0xRajeev


# Vulnerability details

## Impact

The sweepToken() function in the original Compound code whose specified purpose was to recover accidentally sent ERC20 tokens to contract has been removed. 

The original code comment says: “A public function to sweep accidental ERC-20 transfers to this contract. Tokens are sent to admin (timelock).” This safety measure is helpful given the number/value of accidentally stuck tokens that are sent to contracts by mistake.

Tokens accidentally sent to this contract will be stuck leading to fund loss for sender.

## Proof of Concept

https://github.com/compound-finance/compound-protocol/blob/b9b14038612d846b83f8a009a82c38974ff2dcfe/contracts/CErc20.sol#L112-L120

https://github.com/code-423n4/2021-04-basedloans/blob/5c8bb51a3fdc334ea0a68fd069be092123212020/code/contracts/CErc20.sol#L109-L121

## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Retain this function unless there is a specific reason to remove it here.

