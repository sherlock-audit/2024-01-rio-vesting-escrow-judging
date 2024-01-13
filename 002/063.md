Cheery Carob Gecko

medium

# Calls to `revokeAll()` can be front-run

## Summary

Recipients can watch the mempools and front-run owner calls to `revokeAll()`


## Vulnerability Detail

There is no time delay between when a user claims and when they get their tokens. If a user watches the mempool (or hires a company that has keepers which do this for them), they can front-run any call to `revokeAll()`.


## Impact

Tokens that should have gone to the owner, go to the recipient instead.


## Code Snippet

No time delay:
```solidity
// File: src/VestingEscrow.sol : VestingEscrow.claim()   #1

133        /// @notice Claim tokens which have vested.
134        /// @param beneficiary Address to transfer claimed tokens to.
135        /// @param amount Amount of tokens to claim.
136        function claim(address beneficiary, uint256 amount) external onlyRecipient returns (uint256) {
137            uint256 claimable = Math.min(unclaimed(), amount);
138            totalClaimed += claimable;
139    
140 @>         token().safeTransfer(beneficiary, claimable);
141            emit Claim(beneficiary, claimable);
142    
143            return claimable;
144:       }
```
https://github.com/sherlock-audit/2024-01-rio-vesting-escrow/blob/main/rio-vesting-escrow/src/VestingEscrow.sol#L133-L144


## Tool used

Manual Review


## Recommendation

Add another immutable argument for a time delay, and require that a user call a separate function to initiate the claim

