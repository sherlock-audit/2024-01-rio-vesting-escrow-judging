Sunny Grey Zebra

medium

# `recoverERC20` function will fail if the recipient address gets blacklisted in `VestingEscrow.sol`

## Summary
```recoverERC20``` function does not work if the ```recipient```  address gets blacklisted
## Vulnerability Detail
Some ERC-20 tokens, for example, USDC (which is used by the protocol) have the functionality to blacklist specific addresses, so that they are not allowed to transfer and receive tokens. Sending funds to these addresses will lead to a revert. 
The protocol claims to recover all types of ERC20 tokens. However, if the recipient gets blacklisted by USDC then the recipient will not be able to withdraw or recover these ERC20 tokens, and these tokens are stuck in the contract. 
Hence, the ```recoverERC20``` function will not work, and any USDC that is sent to this contract by anyone either intentionally or unintentionally (accidentally) will be locked.




## Impact
Any USDC that is sent to this contract will be stuck and there is no way to recover this. 
## Code Snippet
This is the instance of protocol claiming to recover all types of ERC20 tokens: 

https://github.com/sherlock-audit/2024-01-rio-vesting-escrow/blob/main/rio-vesting-escrow/src/VestingEscrow.sol#L199

```solidity
   /// @notice Recover any ERC20 token to the recipient.
```

This is the function to recover all types of ERC20 tokens:

https://github.com/sherlock-audit/2024-01-rio-vesting-escrow/blob/main/rio-vesting-escrow/src/VestingEscrow.sol#L202

```solidity
    function recoverERC20(address token_, uint256 amount) external {
        uint256 recoverable = amount;
        if (token_ == address(token())) {
            uint256 available = token().balanceOf(address(this)) - (locked() + unclaimed());
            recoverable = Math.min(recoverable, available);
        }
        if (recoverable > 0) {
            IERC20(token_).safeTransfer(recipient(), recoverable);
            emit ERC20Recovered(token_, recoverable);
        }
    }
```
## Tool used

Manual Review

## Recommendation
Add `onlyRecipient` modifier and allow the recipient to pass a beneficiary address as a parameter for receiving the ERC20 token in the ```recoverERC20``` function similar to the ```claim``` function instead of transferring directly to the recipient address.