## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed

# [Lack of Validation Check](https://github.com/code-423n4/2021-08-gravitybridge-findings/issues/55) 

# Handle

defsec


# Vulnerability details

## Impact

During the manual code review, It has been observed that on the cosmos side Coin amount has not been checked on the token definition. That can use misfunctionality on the bridge.  Although zero amount definition fee will be calculated. That can cause lose of user funds. 

## Proof of Concept

1. Navigate to "https://github.com/althea-net/cosmos-gravity-bridge/blob/main/module/x/gravity/types/ethereum.go" Line #69.
2. On the following code, ValidateBasic function does not validate amount.

```
// ValidateBasic permforms stateless validation
func (e *ERC20Token) ValidateBasic() error {
	if err := ValidateEthAddress(e.Contract); err != nil {
		return sdkerrors.Wrap(err, "ethereum address")
	}
	// TODO: Validate all the things
	return nil
}
```

## Tools Used

## Recommended Mitigation Steps

Add the following validation steps on the ValidationBasic function.

```
	if !m.Amount.IsValid() {
		return cosmos.ErrInvalidCoins("coins must be valid")
	}

	if !m.Amount.IsAllPositive() {
		return cosmos.ErrInvalidCoins("coins must be positive")
	}
```



