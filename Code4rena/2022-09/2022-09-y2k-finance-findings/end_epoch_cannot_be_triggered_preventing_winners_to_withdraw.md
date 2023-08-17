## Tags

- bug
- 3 (High Risk)
- high quality report
- sponsor confirmed
- selected for report

# [End epoch cannot be triggered preventing winners to withdraw](https://github.com/code-423n4/2022-09-y2k-finance-findings/issues/278) 

# Lines of code

https://github.com/code-423n4/2022-09-y2k-finance/blob/2175c044af98509261e4147edeb48e1036773771/src/Controller.sol#L198
https://github.com/code-423n4/2022-09-y2k-finance/blob/2175c044af98509261e4147edeb48e1036773771/src/Controller.sol#L246
https://github.com/code-423n4/2022-09-y2k-finance/blob/2175c044af98509261e4147edeb48e1036773771/src/Controller.sol#L261
https://github.com/code-423n4/2022-09-y2k-finance/blob/2175c044af98509261e4147edeb48e1036773771/src/Controller.sol#L277-L286
https://github.com/code-423n4/2022-09-y2k-finance/blob/2175c044af98509261e4147edeb48e1036773771/src/Vault.sol#L203


# Vulnerability details

## Impact
At the end of an epoch, the [triggerEndEpoch(...)](https://github.com/code-423n4/2022-09-y2k-finance/blob/2175c044af98509261e4147edeb48e1036773771/src/Controller.sol#L198) is called to trigger 'epoch end without depeg event', making risk users the winners and entitling them to [withdraw](https://github.com/code-423n4/2022-09-y2k-finance/blob/2175c044af98509261e4147edeb48e1036773771/src/Vault.sol#L203) (risk + hedge) from the vault.
In the case of the Arbitrum sequencer going down or restarting, there is a [grace period of one hour](https://github.com/code-423n4/2022-09-y2k-finance/blob/2175c044af98509261e4147edeb48e1036773771/src/Controller.sol#L285) before the [getLatestPrice()](https://github.com/code-423n4/2022-09-y2k-finance/blob/2175c044af98509261e4147edeb48e1036773771/src/Controller.sol#L261) returns to execute without reverting. This means that the [triggerEndEpoch(...)](https://github.com/code-423n4/2022-09-y2k-finance/blob/2175c044af98509261e4147edeb48e1036773771/src/Controller.sol#L198) cannot complete during this time, because it calls the [getLatestPrice()](https://github.com/code-423n4/2022-09-y2k-finance/blob/2175c044af98509261e4147edeb48e1036773771/src/Controller.sol#L261).

Making this high-priority because unless the [triggerEndEpoch(...)](https://github.com/code-423n4/2022-09-y2k-finance/blob/2175c044af98509261e4147edeb48e1036773771/src/Controller.sol#L198) completes:
- winners cannot [withdraw](https://github.com/code-423n4/2022-09-y2k-finance/blob/2175c044af98509261e4147edeb48e1036773771/src/Vault.sol#L203) althought the epoch is over;
- during this time the strike price might be reached causing a depeg event at all effects turning the table for the winners;
- the [getLatestPrice()](https://github.com/code-423n4/2022-09-y2k-finance/blob/2175c044af98509261e4147edeb48e1036773771/src/Controller.sol#L261) is not functional to the completion of the [triggerEndEpoch(...)](https://github.com/code-423n4/2022-09-y2k-finance/blob/2175c044af98509261e4147edeb48e1036773771/src/Controller.sol#L198), nor to the [withdraw](https://github.com/code-423n4/2022-09-y2k-finance/blob/2175c044af98509261e4147edeb48e1036773771/src/Vault.sol#L203), but only informative used to initialize the event object emitted [at the very end of the triggerEndEpoch function](https://github.com/code-423n4/2022-09-y2k-finance/blob/2175c044af98509261e4147edeb48e1036773771/src/Controller.sol#L246).

First two points each constitute independent jsutification, thrid point reinforces the first 2 points.


## Proof of Concept

### triggerEndEpoch reverts if arbiter down or restarted less than eq GRACE_PERIOD_TIME ago (1hr)

File: [Controller.sol:L246](https://github.com/code-423n4/2022-09-y2k-finance/blob/2175c044af98509261e4147edeb48e1036773771/src/Controller.sol#L246)

Revert if getLatestPrice reverts.

```solidity
function triggerEndEpoch(uint256 marketIndex, uint256 epochEnd) public {
    
    < ... omitted ... >

    emit DepegInsurance(
        keccak256(
            abi.encodePacked(
                marketIndex,
                insrVault.idEpochBegin(epochEnd),
                epochEnd
            )
        ),
        tvl,
        false,
        epochEnd,
        block.timestamp,
        getLatestPrice(insrVault.tokenInsured()) // @audit getLatestPrice reverts while sequencer unavailable or during grace period
    );
}
```

File: [Controller.sol:L277-L286](https://github.com/code-423n4/2022-09-y2k-finance/blob/2175c044af98509261e4147edeb48e1036773771/src/Controller.sol#L277-L286)

Revert if sequencer down or grace period after restart not over.

```solidity
function getLatestPrice(address _token)
    public
    view
    returns (int256 nowPrice)
{
    < ... omitted ... >

    bool isSequencerUp = answer == 0;
    if (!isSequencerUp) {
        revert SequencerDown();
    }

    // Make sure the grace period has passed after the sequencer is back up.
    uint256 timeSinceUp = block.timestamp - startedAt;
    if (timeSinceUp <= GRACE_PERIOD_TIME) { // @audit 1 hour
        revert GracePeriodNotOver();
    }

    < ... omitted ... >
}
```

### withdraw fails if triggerEndEpoch did not execute successfully

File: [Vault.sol:L203](https://github.com/code-423n4/2022-09-y2k-finance/blob/2175c044af98509261e4147edeb48e1036773771/src/Vault.sol#L203)

Can execute if block.timestamp > epochEnd, but fails if trigger did not execute. Winners cannot withdraw.

```solidity
function withdraw(
    uint256 id,
    uint256 assets,
    address receiver,
    address owner
)
    external
    override
    epochHasEnded(id) // @audit same as require((block.timestamp > id) || idDepegged[id]), hence independent from triggers.
    marketExists(id)
    returns (uint256 shares)
{
    < ... omitted ... >

    uint256 entitledShares = beforeWithdraw(id, shares); // @audit ratio is idClaimTVL[id]/ifFinalTVL[id], hence zero unless triggers executed
    
    < ... omitted ... >

    emit Withdraw(msg.sender, receiver, owner, id, assets, entitledShares);
    asset.transfer(receiver, entitledShares);

    return entitledShares;
}
```

## Tools Used
n/a

## Recommended Mitigation Steps

The latest price is retrieved at the very end of the [triggerEndEpoch(...)](https://github.com/code-423n4/2022-09-y2k-finance/blob/2175c044af98509261e4147edeb48e1036773771/src/Controller.sol#L198) for the only purpose of initializing the DepegInsurance event. 
Since it is used for informational purpose (logging / offchain logging) and not for functional purpose to the [triggerEndEpoch(...)](https://github.com/code-423n4/2022-09-y2k-finance/blob/2175c044af98509261e4147edeb48e1036773771/src/Controller.sol#L198) execution, it can be relaxed. 

Depending on how the event is used, when getLatestPrice() is called for informative/logging purpose only, there could be few alternatives:
- log a 0 when SequencerDown or GRACE_PERIOD_TIME not passed
- log a 0 when SequencerDown and ignore GRACE_PERIOD_TIME 
Once events are logged off-chain, some post processing may be used to correct/update the values with accurate data.