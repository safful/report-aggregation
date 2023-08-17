## Tags

- bug
- 2 (Med Risk)
- disagree with severity
- downgraded by judge
- sponsor confirmed

# [`increaseUnlockTime` missing `_checkpoint` for delegated values](https://github.com/code-423n4/2022-08-fiatdao-findings/issues/318) 

# Lines of code

https://github.com/code-423n4/2022-08-fiatdao/blob/fece3bdb79ccacb501099c24b60312cd0b2e4bb2/contracts/VotingEscrow.sol#L509-L515


# Vulnerability details

### [PNM-001] `increaseUnlockTime` missing `_checkpoint` for delegated values.


#### Links

+ https://github.com/code-423n4/2022-08-fiatdao/blob/fece3bdb79ccacb501099c24b60312cd0b2e4bb2/contracts/VotingEscrow.sol#L509-L515

#### Description

In the VotingEscrow contract, users can increase their voting power by:
+ Adding more funds to their delegated valule
+ Increasing the time of their lock
+ Being delegated by another user

Specifically, when users are delegated by other users through the `delegate` function, the delegated user gains control over the delegate funds from the delegating user. 

The delegated user can further increase this power by increasing the time that the delegated funds are locked by calling `increaseUnlockTime`, resulting in ALL the delegated funds controlled by the delegated user, including those that do not originate from the delegated user, being used to increase the voting power of the user.

The issue lies in the following scenario: If user A delegates to user B, and then user B delegates to user C, user B loses the ability to extend his or her voting power by `increaseUnlockTime` due to a missing `_checkpoint` operation. If user B calls the `increaseUnlockTime` function, the `_checkpoint` operation will not proceed, as user B is delegating to user C. However, B still owns delegated funds, in the form of the funds delegated from user A. Therefore, user B should still gain voting power from `increaseUnlockTime`, even though user B is delegating.

#### PoC / Attack Scenario

Assume three users, Alice, Bob, and Carol, who each possess `locks` with 10 units of `delegate` value. Also assume that the unlock time is 1 week.

+ Alice delegates her 10 units to Bob.
+ Bob then delegates his 10 units to Carol.
+ At this point, Alice has 0 `delegate`, value, Bob has 10 `delegate` value, and Carol has 20 `delegate` value.
+ Carol calls `increaseUnlockTime` to 2 weeks, resulting in `_checkpoint` raising her voting power accordingly.
+ Bob calls `increaseUnlockTime` to 2 weeks, resulting in no change in his voting power, even though he has 10 units of `delegate` value.


#### Suggested Fix

Move the `_checkpoint` outside of the `if` statement on line 514.

---