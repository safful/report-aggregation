## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [All except one Comptroller verify functions do not verify anything in Comptroller.sol/CToken.sol ](https://github.com/code-423n4/2021-04-basedloans-findings/issues/18) 

# Handle

0xRajeev


# Vulnerability details

## Impact

Six of the seven Comptroller verify functions do nothing. Not sure why their calls in CToken.sol have been uncommented from the original Compound version.

Except redeemVerify(), six other verify functions transferVerify(), mintVerify(), borrowVerify(), repayBorrowVerify(), liquidateBorrowVerify() and seizeVerify() have no logic except accessing state variables to not be marked pure. Calls to these functions were commented out in the original Compound code’s CToken.sol but have been uncommented here.

Given that they do not implement any logic, the protocol should not be making any assumptions about any defence provided from their unimplemented verification logic.

## Proof of Concept

Dummy functions whose comments say “// Shh - currently unused”:

https://github.com/code-423n4/2021-04-basedloans/blob/5c8bb51a3fdc334ea0a68fd069be092123212020/code/contracts/Comptroller.sol#L263-L281

https://github.com/code-423n4/2021-04-basedloans/blob/5c8bb51a3fdc334ea0a68fd069be092123212020/code/contracts/Comptroller.sol#L402-L418

https://github.com/code-423n4/2021-04-basedloans/blob/5c8bb51a3fdc334ea0a68fd069be092123212020/code/contracts/Comptroller.sol#L450-L474

https://github.com/code-423n4/2021-04-basedloans/blob/5c8bb51a3fdc334ea0a68fd069be092123212020/code/contracts/Comptroller.sol#L519-L546

https://github.com/code-423n4/2021-04-basedloans/blob/5c8bb51a3fdc334ea0a68fd069be092123212020/code/contracts/Comptroller.sol#L584-L609

https://github.com/code-423n4/2021-04-basedloans/blob/5c8bb51a3fdc334ea0a68fd069be092123212020/code/contracts/Comptroller.sol#L638-L656



Uncommented calls from modified code:

https://github.com/code-423n4/2021-04-basedloans/blob/5c8bb51a3fdc334ea0a68fd069be092123212020/code/contracts/CToken.sol#L126

https://github.com/code-423n4/2021-04-basedloans/blob/5c8bb51a3fdc334ea0a68fd069be092123212020/code/contracts/CToken.sol#L560

https://github.com/code-423n4/2021-04-basedloans/blob/5c8bb51a3fdc334ea0a68fd069be092123212020/code/contracts/CToken.sol#L798

https://github.com/code-423n4/2021-04-basedloans/blob/5c8bb51a3fdc334ea0a68fd069be092123212020/code/contracts/CToken.sol#L915

https://github.com/code-423n4/2021-04-basedloans/blob/5c8bb51a3fdc334ea0a68fd069be092123212020/code/contracts/CToken.sol#L1019

https://github.com/code-423n4/2021-04-basedloans/blob/5c8bb51a3fdc334ea0a68fd069be092123212020/code/contracts/CToken.sol#L1090



Commented calls from original Compound code:

https://github.com/compound-finance/compound-protocol/blob/b9b14038612d846b83f8a009a82c38974ff2dcfe/contracts/CToken.sol#L123-L124

https://github.com/compound-finance/compound-protocol/blob/b9b14038612d846b83f8a009a82c38974ff2dcfe/contracts/CToken.sol#L558-L559

https://github.com/compound-finance/compound-protocol/blob/b9b14038612d846b83f8a009a82c38974ff2dcfe/contracts/CToken.sol#L797-L798

https://github.com/compound-finance/compound-protocol/blob/b9b14038612d846b83f8a009a82c38974ff2dcfe/contracts/CToken.sol#L915-L916

https://github.com/compound-finance/compound-protocol/blob/b9b14038612d846b83f8a009a82c38974ff2dcfe/contracts/CToken.sol#L1020-L1021

https://github.com/compound-finance/compound-protocol/blob/b9b14038612d846b83f8a009a82c38974ff2dcfe/contracts/CToken.sol#L1092-L1093



## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Add logic to implement verification if that is indeed assumed to be implemented but is actually not. Otherwise, comment call sites.

