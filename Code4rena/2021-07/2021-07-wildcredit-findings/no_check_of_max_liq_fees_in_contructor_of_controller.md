## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [No check of MAX_LIQ_FEES in contructor of Controller](https://github.com/code-423n4/2021-07-wildcredit-findings/issues/24) 

# Handle

gpersoon


# Vulnerability details

## Impact
Both the functions setLiqParamsToken and setLiqParamsDefault have a check to make sure that
_liqFeeCaller + _liqFeeSystem <= MAX_LIQ_FEES

However the constructor of Controller sets the same parameters and doesn't have this check.
It seems logical to also do the check in the controller otherwise the parameters could be set outside of the wanted range.

## Proof of Concept
// https://github.com/code-423n4/2021-07-wildcredit/blob/main/contracts/Controller.sol#L49
constructor( address _interestRateModel, uint _liqFeeSystemDefault, uint _liqFeeCallerDefault) {
    ...
    liqFeeSystemDefault = _liqFeeSystemDefault;
    liqFeeCallerDefault = _liqFeeCallerDefault;

function setLiqParamsToken( address _token, uint    _liqFeeSystem, uint    _liqFeeCaller ) external onlyOwner {
    require(_liqFeeCaller + _liqFeeSystem <= MAX_LIQ_FEES, "Controller: fees too high");
...
    liqFeeSystemToken[_token] = _liqFeeSystem;
    liqFeeCallerToken[_token] = _liqFeeCaller;

function setLiqParamsDefault( uint    _liqFeeSystem, uint    _liqFeeCaller) external onlyOwner {
    require(_liqFeeCaller + _liqFeeSystem <= MAX_LIQ_FEES, "Controller: fees too high");
    liqFeeSystemDefault = _liqFeeSystem;
    liqFeeCallerDefault = _liqFeeCaller;

## Tools Used

## Recommended Mitigation Steps
Add something like the following in the constructor of Controller
 require(liqFeeCallerDefault + liqFeeSystemDefault <= MAX_LIQ_FEES, "Controller: fees too high");

