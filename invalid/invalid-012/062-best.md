Cheery Carob Gecko

medium

# Escrow recipients can steal the vesting tokens if they have multiple entrypoints

## Summary

The [`recoverERC20()`](https://github.com/sherlock-audit/2024-01-rio-vesting-escrow/blob/main/rio-vesting-escrow/src/VestingEscrow.sol#L202) function will allow escrow recipients to steal their tokens early.


## Vulnerability Detail

One of the protocol invariants states the following about escrow recipients:
```markdown
This role should not be able to claim tokens before they're vested.
```
https://github.com/sherlock-audit/2024-01-rio-vesting-escrow/blob/main/README.md?plain=1#L51

In cases where tokens have multiple entry points, or cases where they gain them due to upgrades/bugs, users will be able to sweep their tokens. There have been [cases](https://blog.openzeppelin.com/compound-tusd-integration-issue-retrospective/) in the past where a token mistakenly had two addresses that could control its balance, and transfers using one address impacted the balance of the other.


## Impact

Broken invariant, and funds taken when they shouldn't have been.


## Code Snippet

There are no balance checks of the `token()` if the normal address of the token isn't used:
```solidity
// File: src/VestingEscrow.sol : VestingEscrow.recoverERC20()   #1

199        /// @notice Recover any ERC20 token to the recipient.
200        /// @param token_ Address of the ERC20 token to recover.
201        /// @param amount Amount of tokens to recover.
202        function recoverERC20(address token_, uint256 amount) external {
203            uint256 recoverable = amount;
204            if (token_ == address(token())) {
205                uint256 available = token().balanceOf(address(this)) - (locked() + unclaimed());
206                recoverable = Math.min(recoverable, available);
207            }
208            if (recoverable > 0) {
209                IERC20(token_).safeTransfer(recipient(), recoverable);
210                emit ERC20Recovered(token_, recoverable);
211            }
212:       }
```
https://github.com/sherlock-audit/2024-01-rio-vesting-escrow/blob/main/rio-vesting-escrow/src/VestingEscrow.sol#L199-L212


## Tool used

Manual Review


## Recommendation

Measure the escrow's token balance before and after the transfer, and revert if it decreases
