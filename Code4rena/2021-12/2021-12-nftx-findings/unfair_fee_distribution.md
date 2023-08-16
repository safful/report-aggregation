## Tags

- bug
- 1 (Low Risk)
- disagree with severity
- resolved
- sponsor confirmed

# [Unfair fee distribution](https://github.com/code-423n4/2021-12-nftx-findings/issues/108) 

# Handle

p4st13r4


# Vulnerability details

## Impact

Detailed description of the impact of this finding.

Fee distribution algorithm in `NFTXSimpleFeeDistributor.sol` can led to unfair distribution of fees among receivers. The `_sendForReceiver` function returns `success = true` only when the call to receiver' `receiveRewards` is successful and the whole amount is transfered to the receiver. Otherwise, the entire fee is moved up to the next receiver, and finally to the treasury.

If a badly implemented receiver leaves a part of the fee (even 1 wei) to the fee distributor, the operation is considered unsuccessful and the entire amount of the fee is moved up to the next receiver. This could lead to the situation where one of the late receivers is unable to receive any fee at all, since some previous receiver has received more than it should have.

## Proof of Concept

[https://github.com/code-423n4/2021-12-nftx/blob/main/nftx-protocol-v2/contracts/solidity/NFTXSimpleFeeDistributor.sol#L166](https://github.com/code-423n4/2021-12-nftx/blob/main/nftx-protocol-v2/contracts/solidity/NFTXSimpleFeeDistributor.sol#L166)

[https://github.com/code-423n4/2021-12-nftx/blob/main/nftx-protocol-v2/contracts/solidity/NFTXSimpleFeeDistributor.sol#L69](https://github.com/code-423n4/2021-12-nftx/blob/main/nftx-protocol-v2/contracts/solidity/NFTXSimpleFeeDistributor.sol#L69)

## Tools Used

Editor

## Recommended Mitigation Steps

`_sendForReceiver` should return a tuple: `(bool success, uint256 amountLeft)`. Then `amountLeft` should be used by `distribute` for the `leftover` variable

