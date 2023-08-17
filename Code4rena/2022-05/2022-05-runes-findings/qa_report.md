## Tags

- bug
- QA (Quality Assurance)
- sponsor confirmed

# [QA Report](https://github.com/code-423n4/2022-05-runes-findings/issues/102) 

# Non Critical
## [N01] Delete `payable` from `withdrawAll()`:
`withdrawAll()` do not use `msg.value` and just withdraw eth so it doesn't need to be `payable`.
### Recommended Mitigation Steps:
```ForgottenRunesWarriorsGuild.sol:163
  - function withdrawAll() public payable onlyOwner {
  + function withdrawAll() public onlyOwner {

ForgottenRunesWarriorsMinter.sol:616
  - function withdrawAll() public payable onlyOwner {
  + function withdrawAll() public onlyOwner {
```

## [N02] Check `.transfer()` return's value:
It is good to add a `require()` statement that checks the return value of token transfers or to use something like OpenZeppelin’s `safeTransfer` unless one is sure the given token reverts in case of a failure. Failure to do so will cause silent failures.

```
ForgottenRunesWarriorsGuild.sol
  175,9:         token.transfer(msg.sender, amount);

ForgottenRunesWarriorsMinter.sol
  629,9:         token.transfer(msg.sender, amount);

ForgottenRunesWarriorsMinter.sol
  402,13:             IERC20(weth).transfer(to, amount);
```
### Recommended Mitigation Steps
Consider using safeTransfer or check transfer's return with require().

## [N03] `type(uint256).max` is more readable than hex version of the number:
```
ForgottenRunesWarriorsMinter.sol
  18,9:         0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff;
  23,9:         0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff;
  27,9:         0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff;
  31,9:         0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff;
  35,9:         0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff;
```
### Recommended Mitigation Steps:
Use `type(uint256).max` instead of `0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff`.
