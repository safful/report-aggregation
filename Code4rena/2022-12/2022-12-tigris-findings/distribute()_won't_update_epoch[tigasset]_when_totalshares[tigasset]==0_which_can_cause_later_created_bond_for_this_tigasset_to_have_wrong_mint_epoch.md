## Tags

- bug
- 2 (Med Risk)
- selected for report
- sponsor confirmed
- M-16

# [distribute() won't update epoch[tigAsset] when totalShares[tigAsset]==0 which can cause later created bond for this tigAsset to have wrong mint epoch](https://github.com/code-423n4/2022-12-tigris-findings/issues/436) 

# Lines of code

https://github.com/code-423n4/2022-12-tigris/blob/588c84b7bb354d20cbca6034544c4faa46e6a80e/contracts/BondNFT.sol#L206-L228
https://github.com/code-423n4/2022-12-tigris/blob/588c84b7bb354d20cbca6034544c4faa46e6a80e/contracts/BondNFT.sol#L48-L86


# Vulnerability details

## Impact
Function `BondNFT.createLock()` creates a bond and it sets bond's mint epoch as `epoch[asset]`, function `Lock.lock()` first calls `claimGovFees()` which calls `BondNFT.distribute()` for all assets and updates the `epoch[assets]` for all assets. so during normal bond creation the value of `epoch[asset]` would be updated and bond would be created from `today` epoch to `today+period` epoch. but if `totalShares[tigAsset] == 0` for an asset, then `distribute()` won't update `epoch[asset]` for that asset and `epoch[asset]` will be some old epoch(will be the start time where asset is added or the time where `totalShares[_tigAsset] != 0`). This would make `createLock()` to set very wrong value for bond's mint epoch when `totalShares[tigAsset] == 0`.
This would happen for the first bond that has been created for that asset always and it will happen again if for some period `totalShares[asset]` become 0, then the next bond would have wrong mint epoch. or `setAllowedAsset(asset, false)`  has been called for that asset.

## Proof of Concept
This is `distribute()` code in BondNFT contract:
```
function distribute(
        address _tigAsset,
        uint _amount
    ) external {
        if (totalShares[_tigAsset] == 0 || !allowedAsset[_tigAsset]) return;
        IERC20(_tigAsset).transferFrom(_msgSender(), address(this), _amount);
        unchecked {
            uint aEpoch = block.timestamp / DAY;
            if (aEpoch > epoch[_tigAsset]) {
                for (uint i=epoch[_tigAsset]; i<aEpoch; i++) {
                    epoch[_tigAsset] += 1;
                    accRewardsPerShare[_tigAsset][i+1] = accRewardsPerShare[_tigAsset][i];
                }
            }
            accRewardsPerShare[_tigAsset][aEpoch] += _amount * 1e18 / totalShares[_tigAsset];
        }
        emit Distribution(_tigAsset, _amount);
    }
```
As you can see when `totalShares[_tigAsset] == 0` then the value of `epoch[_tigAsset]` won't get updated to the today. and there is no other logics in the code to update `epoch[tigAsset]`. so when `totalShares[_tigAsset] == 0` then the value of the `epoch[tigAsset]` would be out dated. this would happen when asset is recently added to the BondNFT assets or when in some time there is no bond left.
When this condition happens and a user call `Lock.lock()` to create a bond the `lock()` function would call `claimGovFees()` to update rewards in BondNFT but because for that asset the value of totalShares are 0 so for that asset `epoch[]` won't get updated and in the `BondNFT.createLock()` the wrong value would set as bond't mint epoch.
This is `Lock.lock()` code:
```
    function lock(
        address _asset,
        uint _amount,
        uint _period
    ) public {
        require(_period <= maxPeriod, "MAX PERIOD");
        require(_period >= minPeriod, "MIN PERIOD");
        require(allowedAssets[_asset], "!asset");

        claimGovFees();

        IERC20(_asset).transferFrom(msg.sender, address(this), _amount);
        totalLocked[_asset] += _amount;
        
        bondNFT.createLock( _asset, _amount, _period, msg.sender);
    }
```
And this is `BondNFT.createLock()` code:
```
    function createLock(
        address _asset,
        uint _amount,
        uint _period,
        address _owner
    ) external onlyManager() returns(uint id) {
        require(allowedAsset[_asset], "!Asset");
        unchecked {
            uint shares = _amount * _period / 365;
            uint expireEpoch = epoch[_asset] + _period;
            id = ++totalBonds;
            totalShares[_asset] += shares;
            Bond memory _bond = Bond(
                id,             // id
                address(0),     // owner
                _asset,         // tigAsset token
                _amount,        // tigAsset amount
                epoch[_asset],  // mint epoch
                block.timestamp,// mint timestamp
                expireEpoch,    // expire epoch
                0,              // pending
                shares,         // linearly scaling share of rewards
                _period,        // lock period
                false           // is expired boolean
            );
            _idToBond[id] = _bond;
            _mint(_owner, _bond);
        }
        emit Lock(_asset, _amount, _period, _owner, id);
    }
```

if a bond get wrong value for mint epoch it would have wrong value for expire epoch and user would get a lot of share by lock for small time. for example this scenario:
1. let's assume `epoch[asset1]` is out dated and it shows 30 days ago epoch. (`allowedAsset[asset1]` was false so locking was not possible and then is set as true after 30 days)
2. during this time because `totalShare[asset1]` was 0 so `distribute()` function won't udpate `epoch[asset1]` and `epoch[asset1]` would show 30 days ago.
3. attacker would create a lock for 32 days by calling `Lock.lock(asset1)`. code would call `BondNFT.createLock()` and would create a bond for attacker which epoch start time is 30 days ago and epoch expire time is 2 days later and attacker receives shares for 32 days.
4. some reward would get distributed into the BondNFT for the `asset1`.
5. other users would create lock too.
6. attacker would claim his rewards and his rewards would be for 32 day locking but attacker lock his tokens for 2 days in reality.

so attacker was able to create lock for long time and get shares and rewards based on that but attacker can release lock after short time.

## Tools Used
VIM

## Recommended Mitigation Steps
update `epoch[asset]` in `distribute()` function  even when `totalShares[_tigAsset]` is equal to 0. only the division by zero and fund transfer should be prevented when totalShare is zero and `epoch[asset]` index should be updated.