## Tags

- bug
- QA (Quality Assurance)
- resolved
- sponsor confirmed

# [QA Report](https://github.com/code-423n4/2022-04-backed-findings/issues/134) 

## Low Risk Issues

### Loans can be created and paid with non-existent/destructed tokens
`@rari-capital/solmate/src/utils/SafeTransferLib.sol` has functions named similarly to functions that OpenZeppelin has, but they act differently. At the top of the file is the following comment:
```solidity
/// @dev Note that none of the functions in this library check that a token has code at all! That responsibility is delegated to the caller.
```
https://github.com/Rari-Capital/solmate/blob/4eaf6b68202e36f67cab379768ac6be304c8ebde/src/utils/SafeTransferLib.sol#L9

If the caller of these functions does not check that the token has code, calls to these functions will be no-ops, since low level calls to non-contracts always return success. There are many instances of these calls throughout the file with no code existence checks:
```
contracts/NFTLoanFacilitator.sol:155:            ERC20(loanAssetContractAddress).safeTransferFrom(msg.sender, address(this), amount);
contracts/NFTLoanFacilitator.sol:157:            ERC20(loanAssetContractAddress).safeTransfer(
contracts/NFTLoanFacilitator.sol:200:                ERC20(loanAssetContractAddress).safeTransferFrom(
contracts/NFTLoanFacilitator.sol:205:                ERC20(loanAssetContractAddress).safeTransfer(
contracts/NFTLoanFacilitator.sol:210:                ERC20(loanAssetContractAddress).safeTransfer(
contracts/NFTLoanFacilitator.sol:215:                ERC20(loan.loanAssetContractAddress).safeTransferFrom(
contracts/NFTLoanFacilitator.sol:241:        ERC20(loan.loanAssetContractAddress).safeTransferFrom(msg.sender, lender, interest + loan.loanAmount);
contracts/NFTLoanFacilitator.sol:242:        IERC721(loan.collateralContractAddress).safeTransferFrom(
contracts/NFTLoanFacilitator.sol:262:        IERC721(loan.collateralContractAddress).safeTransferFrom(
contracts/NFTLoanFacilitator.sol:297:        ERC20(asset).safeTransfer(to, amount);
```

### `originationFeeRate`s of less than 1000 may charge no fees if amounts are small
1. File: contracts/NFTLoanFacilitator.sol (line [156](https://github.com/code-423n4/2022-04-backed/blob/e8015d7c4b295af131f017e646ba1b99c8f608f0/contracts/NFTLoanFacilitator.sol#L156))
```solidity
            uint256 facilitatorTake = amount * originationFeeRate / SCALAR;
```
Add a `require()` for `facilitatorTake` to be non-zero if `originationFeeRate` is non-zero, or state the fee logic for small amounts

### A malicious owner can keep the fee rate at zero, but if a large value transfer enters the mempool, the owner can jack the rate up to the maximum
1. File: contracts/NFTLoanFacilitator.sol (lines [306-312](https://github.com/code-423n4/2022-04-backed/blob/e8015d7c4b295af131f017e646ba1b99c8f608f0/contracts/NFTLoanFacilitator.sol#L306-L312))
```solidity
    function updateOriginationFeeRate(uint32 _originationFeeRate) external onlyOwner {
        require(_originationFeeRate <= 5 * (10 ** (INTEREST_RATE_DECIMALS - 2)), "NFTLoanFacilitator: max fee 5%");
        
        originationFeeRate = _originationFeeRate;

        emit UpdateOriginationFeeRate(_originationFeeRate);
    }
```
Store the fee rate during loan creation, along with the maximum fee rate the user will allow, and update to the new rate for that particular loan only when loans are bought out

### A malicious owner can set an effectively infinite improvement rate with `type(uint256).max` after he/she has entered into a loan to prevent others from buying them out
1. File: contracts/NFTLoanFacilitator.sol (lines [320-326](https://github.com/code-423n4/2022-04-backed/blob/e8015d7c4b295af131f017e646ba1b99c8f608f0/contracts/NFTLoanFacilitator.sol#L320-L326))
```solidity
    function updateRequiredImprovementRate(uint256 _improvementRate) external onlyOwner {
        require(_improvementRate > 0, 'NFTLoanFacilitator: 0 improvement rate');

        requiredImprovementRate = _improvementRate;

        emit UpdateRequiredImprovementRate(_improvementRate);
    }
```
Have a sane upper limit to the improvement rate, and don't allow it to change as above

### `tokenURI()` reverts for tokens that don't implement `IERC20Metadata`
While the ticket descriptors are not in scope, the code calling them is. `NFTLoanTicket.tokenURI()`, which is in scope, ends up calling descriptor code which casts the asset to `IERC20Metadata`. This interface is separate from `IERC20` because EIP-20 does not require those functions to exist. If a valid ERC20 token does not implement this interface, casting it and attempting to call non-existant functions will cause the code to revert, which will cause `tokenURI()` to revert.
https://github.com/code-423n4/2022-04-backed/blob/d34ddbdaf8d1bc1bf17446df830db629ee551308/contracts/descriptors/libraries/PopulateSVGParams.sol#L65
https://github.com/code-423n4/2022-04-backed/blob/d34ddbdaf8d1bc1bf17446df830db629ee551308/contracts/descriptors/libraries/PopulateSVGParams.sol#L69
https://github.com/code-423n4/2022-04-backed/blob/d34ddbdaf8d1bc1bf17446df830db629ee551308/contracts/descriptors/libraries/PopulateSVGParams.sol#L83
Use [`safeDecimals()`](https://github.com/boringcrypto/BoringSolidity/blob/ccb743d4c3363ca37491b87c6c9b24b1f5fa25dc/contracts/libraries/BoringERC20.sol#L33-L55) etc

### `_safeMint()` should be used rather than `_mint()` wherever possible
`_mint()` is [discouraged](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/d4d8d2ed9798cc3383912a23b5e8d5cb602f7d4b/contracts/token/ERC721/ERC721.sol#L271) in favor of `_safeMint()` which ensures that the recipient is either an EOA or implements `IERC721Receiver`. Both open [OpenZeppelin](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/d4d8d2ed9798cc3383912a23b5e8d5cb602f7d4b/contracts/token/ERC721/ERC721.sol#L238-L250) and [solmate](https://github.com/Rari-Capital/solmate/blob/4eaf6b68202e36f67cab379768ac6be304c8ebde/src/tokens/ERC721.sol#L180) have versions of this function so that NFTs aren't lost if they're minted to contracts that cannot transfer them back out.

1. File: contracts/NFTLoanTicket.sol (line [34](https://github.com/code-423n4/2022-04-backed/blob/e8015d7c4b295af131f017e646ba1b99c8f608f0/contracts/NFTLoanTicket.sol#L34))
```solidity
        _mint(to, tokenId);
```

### `loanFacilitatorTransfer()` does not verify that the receiver is capable of handling an NFT
EIP-721 states:
```solidity
    /// @notice Transfer ownership of an NFT -- THE CALLER IS RESPONSIBLE
    ///  TO CONFIRM THAT `_to` IS CAPABLE OF RECEIVING NFTS OR ELSE
    ///  THEY MAY BE PERMANENTLY LOST
```
https://github.com/ethereum/EIPs/blob/904be2534386631358766607f4a098e11a401e95/EIPS/eip-721.md?plain=1#L103-L105

The code below was copied from `transferFrom()`, so any function calling `_transfer()` needs to confirm that `to` is capable of receiving NFTs. `loanFacilitatorTransfer()` calls `_transfer()` without completing this check, which can lead to the loss of NFTs. Checking if the address is zero or not is not sufficient; it needs the other checks in [`safeTransferFrom()`](https://github.com/Rari-Capital/solmate/blob/4eaf6b68202e36f67cab379768ac6be304c8ebde/src/tokens/ERC721.sol#L105-L110).
1. File: contracts/LendTicket.sol (lines [24-34](https://github.com/code-423n4/2022-04-backed/blob/e8015d7c4b295af131f017e646ba1b99c8f608f0/contracts/LendTicket.sol#L24-L34))
```solidity
    /// @dev exact copy of 
    /// https://github.com/Rari-Capital/solmate/blob/main/src/tokens/ERC721.sol#L69-L96
    /// with L78 - L81 removed to enable loanFacilitatorTransfer
    function _transfer(
        address from,
        address to,
        uint256 id
    ) internal {
        require(from == ownerOf[id], "WRONG_FROM");

        require(to != address(0), "INVALID_RECIPIENT");
```

### Missing checks for `address(0x0)` when assigning values to `address` state variables

1. File: contracts/NFTLoanFacilitator.sol (line [282](https://github.com/code-423n4/2022-04-backed/blob/e8015d7c4b295af131f017e646ba1b99c8f608f0/contracts/NFTLoanFacilitator.sol#L282))
```solidity
        lendTicketContract = _contract;
```
2. File: contracts/NFTLoanFacilitator.sol (line [292](https://github.com/code-423n4/2022-04-backed/blob/e8015d7c4b295af131f017e646ba1b99c8f608f0/contracts/NFTLoanFacilitator.sol#L292))
```solidity
        borrowTicketContract = _contract;
```

## Non-critical Issues


### `constant`s should be defined rather than using magic numbers

1. File: contracts/NFTLoanFacilitator.sol (line [307](https://github.com/code-423n4/2022-04-backed/blob/e8015d7c4b295af131f017e646ba1b99c8f608f0/contracts/NFTLoanFacilitator.sol#L307))
```solidity
        require(_originationFeeRate <= 5 * (10 ** (INTEREST_RATE_DECIMALS - 2)), "NFTLoanFacilitator: max fee 5%");
```
2. File: contracts/NFTLoanFacilitator.sol (line [384](https://github.com/code-423n4/2022-04-backed/blob/e8015d7c4b295af131f017e646ba1b99c8f608f0/contracts/NFTLoanFacilitator.sol#L384))
```solidity
            * (perAnumInterestRate * 1e18 / 365 days)
```
3. File: contracts/NFTLoanFacilitator.sol (line [384](https://github.com/code-423n4/2022-04-backed/blob/e8015d7c4b295af131f017e646ba1b99c8f608f0/contracts/NFTLoanFacilitator.sol#L384))
```solidity
            * (perAnumInterestRate * 1e18 / 365 days)
```
4. File: contracts/NFTLoanFacilitator.sol (line [385](https://github.com/code-423n4/2022-04-backed/blob/e8015d7c4b295af131f017e646ba1b99c8f608f0/contracts/NFTLoanFacilitator.sol#L385))
```solidity
            / 1e21 // SCALAR * 1e18
```

### Typos

1. File: contracts/NFTLoanFacilitator.sol (line [303](https://github.com/code-423n4/2022-04-backed/blob/e8015d7c4b295af131f017e646ba1b99c8f608f0/contracts/NFTLoanFacilitator.sol#L303))
```solidity
     * @notice Updates originationFeeRate the faciliator keeps of each loan amount
```
faciliator

2. File: contracts/interfaces/INFTLoanFacilitator.sol (line [65](https://github.com/code-423n4/2022-04-backed/blob/e8015d7c4b295af131f017e646ba1b99c8f608f0/contracts/interfaces/INFTLoanFacilitator.sol#L65))
```solidity
     * @param minLoanAmount mimimum loan amount
```
mimimum


### NatSpec is incomplete

1. File: contracts/interfaces/INFTLoanFacilitator.sol (lines [286-288](https://github.com/code-423n4/2022-04-backed/blob/e8015d7c4b295af131f017e646ba1b99c8f608f0/contracts/interfaces/INFTLoanFacilitator.sol#L286-L288))
```solidity
     * @param loanId The loan id
     */
    function totalOwed(uint256 loanId) view external returns (uint256);
```
Missing: `@return` 

2. File: contracts/interfaces/INFTLoanFacilitator.sol (lines [292-294](https://github.com/code-423n4/2022-04-backed/blob/e8015d7c4b295af131f017e646ba1b99c8f608f0/contracts/interfaces/INFTLoanFacilitator.sol#L292-L294))
```solidity
     * @param loanId The loan id
     */
    function interestOwed(uint256 loanId) view external returns (uint256);
```
Missing: `@return` 

3. File: contracts/interfaces/INFTLoanFacilitator.sol (lines [298-300](https://github.com/code-423n4/2022-04-backed/blob/e8015d7c4b295af131f017e646ba1b99c8f608f0/contracts/interfaces/INFTLoanFacilitator.sol#L298-L300))
```solidity
     * @param loanId The loan id
     */
    function loanEndSeconds(uint256 loanId) view external returns (uint256);
```
Missing: `@return` 


### Event is missing `indexed` fields
Each `event` should use three `indexed` fields if there are three or more fields

1. File: contracts/interfaces/INFTLoanFacilitator.sol (lines [68-77](https://github.com/code-423n4/2022-04-backed/blob/e8015d7c4b295af131f017e646ba1b99c8f608f0/contracts/interfaces/INFTLoanFacilitator.sol#L68-L77))
```solidity
    event CreateLoan(
        uint256 indexed id,
        address indexed minter,
        uint256 collateralTokenId,
        address collateralContract,
        uint256 maxInterestRate,
        address loanAssetContract,
        uint256 minLoanAmount,
        uint256 minDurationSeconds
        );
```
2. File: contracts/interfaces/INFTLoanFacilitator.sol (lines [93-99](https://github.com/code-423n4/2022-04-backed/blob/e8015d7c4b295af131f017e646ba1b99c8f608f0/contracts/interfaces/INFTLoanFacilitator.sol#L93-L99))
```solidity
    event Lend(
        uint256 indexed id,
        address indexed lender,
        uint256 interestRate,
        uint256 loanAmount,
        uint256 durationSeconds
    );
```
3. File: contracts/interfaces/INFTLoanFacilitator.sol (line [145](https://github.com/code-423n4/2022-04-backed/blob/e8015d7c4b295af131f017e646ba1b99c8f608f0/contracts/interfaces/INFTLoanFacilitator.sol#L145))
```solidity
     event WithdrawOriginationFees(address asset, uint256 amount, address to);
```
4. File: contracts/interfaces/INFTLoanFacilitator.sol (line [152](https://github.com/code-423n4/2022-04-backed/blob/e8015d7c4b295af131f017e646ba1b99c8f608f0/contracts/interfaces/INFTLoanFacilitator.sol#L152))
```solidity
     event UpdateOriginationFeeRate(uint32 feeRate);
```
5. File: contracts/interfaces/INFTLoanFacilitator.sol (line [159](https://github.com/code-423n4/2022-04-backed/blob/e8015d7c4b295af131f017e646ba1b99c8f608f0/contracts/interfaces/INFTLoanFacilitator.sol#L159))
```solidity
     event UpdateRequiredImprovementRate(uint256 improvementRate);
```