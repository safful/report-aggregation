## Tags

- bug
- 2 (Med Risk)
- downgraded by judge
- judge review requested
- selected for report
- sponsor confirmed
- M-13

# [one can become referral of hash 0x0 and because all users default referral hash is 0x0 so he would become all users referral by default and earn a lot of fees while users didn't approve it](https://github.com/code-423n4/2022-12-tigris-findings/issues/379) 

# Lines of code

https://github.com/code-423n4/2022-12-tigris/blob/588c84b7bb354d20cbca6034544c4faa46e6a80e/contracts/Referrals.sol#L20-L24
https://github.com/code-423n4/2022-12-tigris/blob/588c84b7bb354d20cbca6034544c4faa46e6a80e/contracts/TradingExtension.sol#L148-L152


# Vulnerability details

## Impact
By default the value of `_referred[user]` is 0x0 for all users and if one set 0x0 as his referral hash then he would become referral for all the users who didn't set referral by default and he would earn a lot of referral funds that users didn't approve it.

## Proof of Concept
This is `createReferralCode()` code:
```
    function createReferralCode(bytes32 _hash) external {
        require(_referral[_hash] == address(0), "Referral code already exists");
        _referral[_hash] = _msgSender();
        emit ReferralCreated(_msgSender(), _hash);
    }
```
As you can see attacker can become set 0x0 as his hash referral by calling `createReferralCode(0x0)` and code would set `_referral[0x0] = attackerAddress` (attacker needs to be the first one calling this).
Then in the `getRef()` code the logic would return `attackerAddress` as referral for all the users who didn't set referral.
```
    function getRef(
        address _trader
    ) external view returns(address) {
        return referrals.getReferral(referrals.getReferred(_trader));
    }
```
in the code, getReferred(trader) would return 0x0 because trader didn't set referred and getReferral(0x0) would return attackerAddress.
`_handleOpenFees()` and `_handleCloseFees()` function in the Trading contract would use `getRef(trader)` and they would transfer referral fee to attackerAddress and attacker would receive fee form a lot of users which didn't set any referral, those users didn't set any referral and didn't approve attacker receiving referral fees from them and because most of the users wouldn't know about this and referral codes so attacker would receive a lot of funds.

## Tools Used
VIM

## Recommended Mitigation Steps
prevent some one from setting 0x0 hash for their referral code.