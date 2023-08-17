## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [ AccountantDelegate: sweepInterest function will destroy the cnote in the contract.](https://github.com/code-423n4/2022-06-canto-findings/issues/89) 

# Lines of code

https://github.com/Plex-Engineer/lending-market/blob/ab31a612be354e252d72faead63d86b844172761/contracts/Accountant/AccountantDelegate.sol#L74-L92


# Vulnerability details

## Impact
When the user borrows note tokens, the AccountantDelegate contract provides note tokens and gets cnote tokens. Later, when the user repays the note tokens, the cnote tokens are destroyed and the note tokens are transferred to the AccountantDelegate contract.
However, in the sweepInterest function of the AccountantDelegate contract, all cnote tokens in the contract will be transferred to address 0. This will prevent the user from repaying the note tokens, and the sweepInterest function will not calculate the interest correctly later.
## Proof of Concept
https://github.com/Plex-Engineer/lending-market/blob/ab31a612be354e252d72faead63d86b844172761/contracts/Accountant/AccountantDelegate.sol#L74-L92
https://github.com/Plex-Engineer/lending-market/blob/ab31a612be354e252d72faead63d86b844172761/contracts/CToken.sol#L533
## Tools Used
None
## Recommended Mitigation Steps
```
    function sweepInterest() external override returns(uint) {
		
		uint noteBalance = note.balanceOf(address(this));
		uint CNoteBalance = cnote.balanceOf(address(this));

		Exp memory expRate = Exp({mantissa: cnote.exchangeRateStored()}); // obtain exchange Rate from cNote Lending Market as a mantissa (scaled by 1e18)
		uint cNoteConverted = mul_ScalarTruncate(expRate, CNoteBalance); //calculate truncate(cNoteBalance* mantissa{expRate})
		uint noteDifferential = sub_(note.totalSupply(), noteBalance); //cannot underflow, subtraction first to prevent against overflow, subtraction as integers

		require(cNoteConverted >= noteDifferential, "Note Loaned to LendingMarket must increase in value");
		
		uint amtToSweep = sub_(cNoteConverted, noteDifferential);

		note.transfer(treasury, amtToSweep);

-		cnote.transfer(address(0), CNoteBalance);

		return 0;
    }
```

