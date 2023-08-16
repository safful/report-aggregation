## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed
- resolved

# [Add comment to not obvious code in withdrawDeposit ](https://github.com/code-423n4/2021-06-realitycards-findings/issues/30) 

# Handle

gpersoon


# Vulnerability details

## Impact
In the function withdrawDeposit of RCTreasury.sol, the value of isForeclosed[_msgSender]  is set to true.
In the next statement it is overwritten with a new value. So the first statement seem redundant.
However this is not the case because it is retrieved from the function removeUserFromOrderbook
(see proof of concept below)

As this is not obvious it is probably useful to add a comment so future developers can understand this.

## Proof of Concept
// https://github.com/code-423n4/2021-06-realitycards/blob/main/contracts/RCTreasury.sol#L322
function withdrawDeposit(uint256 _amount, bool _localWithdrawal)  external  override  balancedBooks  {
  ... 
    isForeclosed[_msgSender] = true;   // this seems to be redundant
    isForeclosed[_msgSender] = orderbook.removeUserFromOrderbook( _msgSender );

// https://github.com/code-423n4/2021-06-realitycards/blob/main/contracts/RCOrderbook.sol#L575
  function removeUserFromOrderbook(address _user)  external override returns (bool _userForeclosed) {
        require(treasury.isForeclosed(_user), "User must be foreclosed");   // this checks the isForeclosed value from the treasury contract 

## Tools Used

## Recommended Mitigation Steps
Add a comment to  isForeclosed[_msgSender] = true;  explaining this line is important.


