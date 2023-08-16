## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Router.sol: Optimise calculation of totalTradeFees in addTradeFee()](https://github.com/code-423n4/2021-07-spartan-findings/issues/36) 

# Handle

hickuphh3


# Vulnerability details

### Impact

In the case where `arrayFeeLength < arrayFeeSize`, totalTradeFees is not calculated, so normalAverageFee will be 0. Hence, a return statement can be added to exit the function. Otherwise, when `arrayFeeSize >= arrayFeeLength`, the feeArray elements are iterated through twice:

- First, in `addFee`, to shift the elements by 1 to make way for the new fee. Note that `addFee()` is also solely called by `addTradeFee()`
- Second, for the calculation of totalTradeFees

With all these in mind, we can make the second iteration redundant by combining the total trade fee calculation in `addFee()`.

### Recommended Mitigation Steps

```jsx
function addTradeFee(uint _fee) internal {
	uint arrayFeeLength = feeArray.length;
  if(arrayFeeLength < arrayFeeSize){
		feeArray.push(_fee); // Build array until it is == arrayFeeSize
    return;
   }
	 // If array is required length; shift in place of oldest item
	 // Calculate totalTradeFee at the same time
	 uint totalTradeFees = addCurrentFeeAndCalcTotalTradeFees(arrayFeeLength, _fee); 
	 normalAverageFee = totalTradeFees / arrayFeeSize; // Calc average fee
}

function addCurrentFeeAndCalcTotalTradeFees(
  uint arrayFeeLength,
	uint _fee
) internal returns (uint totalTradeFees) {
	totalTradeFees = _fee; // add newest fee
  // store and update in memory first, for gas optimization
  uint[] memory _feeArray = feeArray;
  for (uint i = arrayFeeLength - 1; i > 0; i--) {
		_feeArray[i] = _feeArray[i - 1];
    totalTradeFees += _feeArray[i];
  }
  _feeArray[0] = _fee;
  feeArray = _feeArray;
}
```

