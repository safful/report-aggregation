## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [minBorrowUSD not initialized in the contract](https://github.com/code-423n4/2021-07-wildcredit-findings/issues/25) 

# Handle

gpersoon


# Vulnerability details

## Impact
The parameter minBorrowUSD of the contract Controller isn't initialized.
If someone is able to Borrow before the function setMinBorrowUSD is called, he might be able to borrow a very small amount.
This might be unwanted.

## Proof of Concept
//https://github.com/code-423n4/2021-07-wildcredit/blob/main/contracts/Controller.sol#L27
  uint public minBorrowUSD;

  function setMinBorrowUSD(uint _value) external onlyOwner {
    minBorrowUSD = _value;
  }

//https://github.com/code-423n4/2021-07-wildcredit/blob/main/contracts/LendingPair.sol#L553
function _checkBorrowLimits(address _token, address _account) internal view {
   ...
    require(accountBorrowUSD >= controller.minBorrowUSD(), "LendingPair: borrow amount below minimum");


## Tools Used

## Recommended Mitigation Steps
Initialize minBorrowUSD via the constructor or set a reasonable default in the contract.

