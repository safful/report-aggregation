## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed
- selected for report
- M-02

# [`BLOCK_PERIOD` is incorrect](https://github.com/code-423n4/2022-10-zksync-findings/issues/259) 

# Lines of code

https://github.com/code-423n4/2022-10-zksync/blob/456078b53a6d09636b84522ac8f3e8049e4e3af5/ethereum/contracts/zksync/Config.sol#L47


# Vulnerability details

The `BLOCK_PERIOD` is set to 13 seconds in `Config.sol`.
```sol
uint256 constant BLOCK_PERIOD = 13 seconds;
```
Since moving to Proof-of-Stake (PoS) after the Merge, block times on ethereum are fixed at 12 seconds per block (slots).
https://ethereum.org/en/developers/docs/consensus-mechanisms/pos/#:~:text=Whereas%20under%20proof%2Dof%2Dwork,block%20proposer%20in%20every%20slot.

### Impact
This results in incorrect calculation of `PRIORITY_EXPIRATION` which is used to determine when a transaction in the Priority Queue should be considered expired.

```sol
uint256 constant PRIORITY_EXPIRATION_PERIOD = 3 days;
/// @dev Expiration delta for priority request to be satisfied (in ETH blocks)
uint256 constant PRIORITY_EXPIRATION = PRIORITY_EXPIRATION_PERIOD/BLOCK_PERIOD;
```
The time difference can be calulated
```python
>>> 3*24*60*60 / 13    # 3 days / 13 sec block period
19938.46153846154
>>> 3*24*60*60 / 12    # 3 days / 12 sec block period
21600.0
>>> 21600 - 19938      # difference in blocks
1662
>>> 1662 * 12 / (60 * 60) # difference in hours
5.54
```
By using block time of 13 seconds, a transaction in the Priority Queue incorrectly expires 5.5 hours earlier than is expected.

5.5 hours is a significant amount of time difference so I believe this issue to be Medium severity.

### Recommendations
Change the block period to be 12 seconds
```sol
uint256 constant BLOCK_PERIOD = 12 seconds;
```
