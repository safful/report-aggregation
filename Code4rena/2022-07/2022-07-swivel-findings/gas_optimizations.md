## Tags

- bug
- duplicate
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Gas Optimizations](https://github.com/code-423n4/2022-07-swivel-findings/issues/78) 

# Summary
| Issue | Instances |
| ------ | :--------: |
| `++i` uses less gas compared to `i++` | 5 |
| `uint8` incurs more gas overhead compared to `uint256` | 7 |
 | Use named return variable instead of using return | 2 |
 | Named return variable is not used | 8 |
 | Use != 0 instead of > 0 for a `uint`  | 17 |
 


# Gas Optimizations

## `++i` uses less gas compared to `i++` 

This is especially relevant for the use of `i++` in `for` loops. This saves 6 gas per loop. 

_There are 5 instances of this issue:_

```
FILE: Swivel/Swivel.sol

100:   unchecked {i++;}
```
https://github.com/code-423n4/2022-07-swivel/blob/main/Swivel/Swivel.sol#L100

```
FILE: Swivel/Swivel.sol

269:    unchecked {i++;}
```
https://github.com/code-423n4/2022-07-swivel/blob/main/Swivel/Swivel.sol#L269

```
FILE: Swivel/Swivel.sol

417      unchecked {
418        i++;
419:      }
```
https://github.com/code-423n4/2022-07-swivel/blob/main/Swivel/Swivel.sol#L417-L419

```
FILE: Swivel/Swivel.sol

510      unchecked {
511        x++;
512:      }
```
https://github.com/code-423n4/2022-07-swivel/blob/main/Swivel/Swivel.sol#L510-L512

```
FILE: Swivel/Swivel.sol

563      unchecked {
564        i++;
565:      }
```
https://github.com/code-423n4/2022-07-swivel/blob/main/Swivel/Swivel.sol#L563-L565


## `uint8`, `uint16`,  incurs more gas overhead compared to `uint256`

_There are 7 instances of this issue:_

```
File: Swivel/Swivel.sol

35:   uint16 constant public MIN_FEENOMINATOR = 33;
```
https://github.com/code-423n4/2022-07-swivel/blob/main/Swivel/Swivel.sol#L35

```
File: Swivel/Swivel.sol

37:   uint16[4] public feenominators;
```
https://github.com/code-423n4/2022-07-swivel/blob/main/Swivel/Swivel.sol#L37

```
FILE: Marketplace/MarketPlace.sol

65:   uint8 p,
```
https://github.com/code-423n4/2022-07-swivel/blob/main/Marketplace/MarketPlace.sol#L65

```
FILE: Creator/Creator.sol

31:   uint8 p,
```
https://github.com/code-423n4/2022-07-swivel/blob/main/Creator/Creator.sol#L31

```
FILE: Creator/Creator.sol

38:   uint8 d
```
https://github.com/code-423n4/2022-07-swivel/blob/main/Creator/Creator.sol#L38

```
FILE: VaultTracker/VaultTracker.sol

26:   uint8 public immutable protocol;
```
https://github.com/code-423n4/2022-07-swivel/blob/main/VaultTracker/VaultTracker.sol#L26

```
FILE: Creator/ZcToken.sol

17:   uint8 public immutable protocol;
```
https://github.com/code-423n4/2022-07-swivel/blob/main/Creator/ZcToken.sol#L17


## Use named return variable instead of using return

_There are 2 instances of this issue:_

```
FILE: Swivel/Swivel.sol

697:  return hash;
```
https://github.com/code-423n4/2022-07-swivel/blob/main/Swivel/Swivel.sol#L697

```
FILE: Swivel/Swivel.sol

207:    return interest;
```
https://github.com/code-423n4/2022-07-swivel/blob/main/Marketplace/MarketPlace.sol#L207


## Named return variable is not used

Remove the named return variable to save 25 gas per function call. 

_There are 8 instances of this issue:_

```
FILE: Marketplace/Marketplace.sol

148:     function authRedeem(uint8 p, address u, uint256 m, address f, address t, uint256 a) public authorized(markets[p][u][m].zcToken) returns (uint256 underlyingAmount) {
```
https://github.com/code-423n4/2022-07-swivel/blob/main/Marketplace/MarketPlace.sol#L148

```
FILE: Creator/ZcToken.sol

43:    function convertToUnderlying(uint256 principalAmount) external override view returns (uint256 underlyingAmount){
```
https://github.com/code-423n4/2022-07-swivel/blob/main/Creator/ZcToken.sol#L43

```
FILE: Creator/ZcToken.sol

52:   function convertToPrincipal(uint256 underlyingAmount) external override view returns (uint256 principalAmount){
```
https://github.com/code-423n4/2022-07-swivel/blob/main/Creator/ZcToken.sol#L52

```
FILE: Creator/ZcToken.sol

61:   function maxRedeem(address owner) external override view returns (uint256 maxPrincipalAmount){
```
https://github.com/code-423n4/2022-07-swivel/blob/main/Creator/ZcToken.sol#L61

```
FILE: Creator/ZcToken.sol

70:    function previewRedeem(uint256 principalAmount) external override view returns (uint256 underlyingAmount){
```
https://github.com/code-423n4/2022-07-swivel/blob/main/Creator/ZcToken.sol#L70

```
FILE: Creator/ZcToken.sol

79:   function maxWithdraw(address owner) external override view returns (uint256 maxUnderlyingAmount){
```
https://github.com/code-423n4/2022-07-swivel/blob/main/Creator/ZcToken.sol#L79

```
FILE: Creator/ZcToken.sol

88:     function previewWithdraw(uint256 underlyingAmount) external override view returns (uint256 principalAmount){
```
https://github.com/code-423n4/2022-07-swivel/blob/main/Creator/ZcToken.sol#L88

```
FILE: Creator/ZcToken.sol

98:     function withdraw(uint256 underlyingAmount, address receiver, address holder) external override returns (uint256 principalAmount){
```
https://github.com/code-423n4/2022-07-swivel/blob/main/Creator/ZcToken.sol#L98

```
FILE: Creator/ZcToken.sol

124:     function redeem(uint256 principalAmount, address receiver, address holder) external override returns (uint256 underlyingAmount){
```
https://github.com/code-423n4/2022-07-swivel/blob/main/Creator/ZcToken.sol#L124


## Use != 0 instead of > 0 for a `uint` 
This saves 6 gas per instance. 

_There are 17 instances of this issue:_

```
FILE: Swivel/Swivel.sol

118:    if (amount > o.premium) { revert Exception(5, amount, o.premium, address(0), address(0)); }
```
https://github.com/code-423n4/2022-07-swivel/blob/main/Swivel/Swivel.sol#L118

```
FILE: Swivel/Swivel.sol

155:   if (amount > o.principal) { revert Exception(5, amount, o.principal, address(0), address(0)); }
```
https://github.com/code-423n4/2022-07-swivel/blob/main/Swivel/Swivel.sol#L155

```
FILE: Swivel/Swivel.sol

190:   if (amount > o.principal) { revert Exception(5, amount, o.principal, address(0), address(0)); }
```
https://github.com/code-423n4/2022-07-swivel/blob/main/Swivel/Swivel.sol#L190

```
FILE: Swivel/Swivel.sol

219:   if (amount > o.premium) { revert Exception(5, amount, o.premium, address(0), address(0)); }
```
https://github.com/code-423n4/2022-07-swivel/blob/main/Swivel/Swivel.sol#L219

```
FILE: Swivel/Swivel.sol

284:  if (amount > o.premium) { revert Exception(5, amount, o.premium, address(0), address(0)); }
```
https://github.com/code-423n4/2022-07-swivel/blob/main/Swivel/Swivel.sol#L284

```
FILE: Swivel/Swivel.sol

315:   if (amount > o.principal) { revert Exception(5, amount, o.principal, address(0), address(0)); }
```
https://github.com/code-423n4/2022-07-swivel/blob/main/Swivel/Swivel.sol#L315

```
FILE: Swivel/Swivel.sol

345:   if (amount > o.principal) { revert Exception(5, amount, o.principal, address(0), address(0)); }
```
https://github.com/code-423n4/2022-07-swivel/blob/main/Swivel/Swivel.sol#L345

```
FILE: Swivel/Swivel.sol

380:   if (amount > o.premium) { revert Exception(5, amount, o.premium, address(0), address(0)); }
```
https://github.com/code-423n4/2022-07-swivel/blob/main/Swivel/Swivel.sol#L380

```
FILE: VaultTracker/VaultTracker.sol

54:    if (vlt.notional > 0) {
```
https://github.com/code-423n4/2022-07-swivel/blob/main/VaultTracker/VaultTracker.sol#L54

```
FILE: VaultTracker/VaultTracker.sol

59:   if (maturityRate > 0) { // Calculate marginal interest
```
https://github.com/code-423n4/2022-07-swivel/blob/main/VaultTracker/VaultTracker.sol#L59

```
FILE: VaultTracker/VaultTracker.sol

86:  if (a > vlt.notional) { revert Exception(31, a, vlt.notional, address(0), address(0)); }
```
https://github.com/code-423n4/2022-07-swivel/blob/main/VaultTracker/VaultTracker.sol#L86

```
FILE: VaultTracker/VaultTracker.sol

93:   if (maturityRate > 0) { // Calculate marginal interest
```
https://github.com/code-423n4/2022-07-swivel/blob/main/VaultTracker/VaultTracker.sol#L93

```
FILE: VaultTracker/VaultTracker.sol

123:   if (maturityRate > 0) { // Calculate marginal interest
```
https://github.com/code-423n4/2022-07-swivel/blob/main/VaultTracker/VaultTracker.sol#L123

```
FILE: VaultTracker/VaultTracker.sol

158:   if (a > from.notional) { revert Exception(31, a, from.notional, address(0), address(0)); }
```
https://github.com/code-423n4/2022-07-swivel/blob/main/VaultTracker/VaultTracker.sol#L158

```
FILE: VaultTracker/VaultTracker.sol

165:   if (maturityRate > 0) { 
```
https://github.com/code-423n4/2022-07-swivel/blob/main/VaultTracker/VaultTracker.sol#L165

```
FILE: VaultTracker/VaultTracker.sol

181:   if (to.notional > 0) {
```
https://github.com/code-423n4/2022-07-swivel/blob/main/VaultTracker/VaultTracker.sol#L181

```
FILE: VaultTracker/VaultTracker.sol

184:   if (maturityRate > 0) { 
```
https://github.com/code-423n4/2022-07-swivel/blob/main/VaultTracker/VaultTracker.sol#L184

```
FILE: VaultTracker/VaultTracker.sol

224:  if (maturityRate > 0) { 
```
https://github.com/code-423n4/2022-07-swivel/blob/main/VaultTracker/VaultTracker.sol#L222

