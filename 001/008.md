Icy Purple Antelope

high

# Ownership Transfer and Unauthorized Recovery

## Summary
The VestingEscrowFactory contract is susceptible to security vulnerabilities, specifically related to ownership transfer and unauthorized recovery of ERC20 tokens and Ether. Unauthorized parties can potentially manipulate the ownership and recovery functions, posing a risk to the overall security and control of the contract.
## Vulnerability Detail
The `recoverERC20` function allows the owner to recover ERC20 tokens, but this operation is not restricted to the owner, enabling potential misuse by an attacker or an unauthorized party. To fix this, add an `onlyOwner` modifier to restrict the recovery functionality to the contract owner.

```solidity
contract VestingEscrowFactory is IVestingEscrowFactory, Ownable2Step {
    // ... existing code

    /// @notice Recover any ERC20 to the owner.
    /// @param token_ The ERC20 token to recover.
    /// @param amount The amount to recover.
    function recoverERC20(address token_, uint256 amount) external onlyOwner {
        if (amount > 0) {
            IERC20(token_).safeTransfer(owner(), amount);
            emit ERC20Recovered(token_, amount);
        }
    }

    // ... existing code
}
```

## Impact
Unauthorized parties could potentially recover ERC20 tokens.
## Code Snippet
https://github.com/sherlock-audit/2024-01-rio-vesting-escrow/blob/main/rio-vesting-escrow/src/VestingEscrowFactory.sol#L80-L85
## Tool used

Manual Review

## Recommendation
Add the `onlyOwner` modifier to the `recoverERC20` function to restrict it to the contract owner