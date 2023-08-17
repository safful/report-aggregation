## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed
- selected for report
- responded

# [`_payoutEth()` calculates `balance` with an offset, always leaving dust `ETH` in the contract](https://github.com/code-423n4/2022-10-holograph-findings/issues/476) 

# Lines of code

https://github.com/code-423n4/2022-10-holograph/blob/f8c2eae866280a1acfdc8a8352401ed031be1373/contracts/enforcer/PA1D.sol#L391
https://github.com/code-423n4/2022-10-holograph/blob/f8c2eae866280a1acfdc8a8352401ed031be1373/contracts/enforcer/PA1D.sol#L395


# Vulnerability details

Payout recipients can call `getEthPayout()` to transfer the ETH balance of the contract to all payout recipients.
This function makes an internal call to `_payoutEth`, which sends the payment to the recipients based on their associated `bp`

The issue is that the `balance` used in the `transfer` calls is not the contract ETH balance, but the balance minus a `gasCost`.

This means `getEthPayout()` calls will leave dust in the contract.

## Impact

If the dust is small enough, a subsequent call to `getEthPayout` is likely to revert because of [this check](https://github.com/code-423n4/2022-10-holograph/blob/f8c2eae866280a1acfdc8a8352401ed031be1373/contracts/enforcer/PA1D.sol#L390).
And `enforcer/PA1D` does not have any other ETH withdrawal function. While `enforcer/PA1D` is meant to be used via delegate calls from a NFT collection contract, if the NFT contract does not have any withdrawal function either, this dust mentioned above is effectively lost.

## Proof-Of-Concept

Let us take the example of a payout recipient trying to retrieve their share of the balance, equal to `40_000` For simplicity, assume one payout address, owned by Alice:

- Alice calls `getEthPayout()`, which in turn calls `_payoutEth()`
- `gasCost = (23300 * length) + length = 23300 + 1 = 23301`
- `balance = address(this).balance = 40000`
- `balance - gasCost = 40000 - 23301 = 16699`,
- `sending = ((bps[i] * balance) / 10000) = 10000 * 16699 / 10000 = 16699`
- Alice receives `16699`.

Alice has to wait for the balance to increase to call `getEthPayout()` again. But no matter what, there will always be at least a dust of `10000` left in the contract.


## Tools Used

Manual Analysis


## Mitigation

The transfers should be done based on `address(this).balance`. The `gasCost` is redundant as the gas amount is specified by the caller of `getEthPayout()`, the contract does not have to provide gas.

```diff
-391: balance = balance - gasCost;
392:     uint256 sending;
393:     // uint256 sent;
394:     for (uint256 i = 0; i < length; i++) {
395:       sending = ((bps[i] * balance) / 10000);
396:       addresses[i].transfer(sending);
397:       // sent = sent + sending;
398:     }
```