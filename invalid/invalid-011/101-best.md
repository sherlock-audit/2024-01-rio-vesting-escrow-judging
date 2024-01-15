Shallow Rose Salamander

medium

# `deployVestingContract()` lacks validation for start time and end time

## Summary

`deployVestingContract()` lacks validation for start time and end time

## Vulnerability Detail
The function `deployVestingContract()` lacks validation for the vesting start and end times. It is possible that the end time is less than the current time, allowing the recipient to claim all tokens before the vesting even begins.
However, in Lido, the [start time](https://github.com/lidofinance/lido-vesting-escrow/blob/main/contracts/VestingEscrowFactory.vy#L93C5) defaults to the time of creation.
```solidity
function deployVestingContract(
        uint256 amount,
        address recipient,
        uint40 vestingDuration,
        uint40 vestingStart,
        uint40 cliffLength,
        bool isFullyRevokable,
        bytes calldata initialDelegateParams
    ) external returns (address escrow) {
        if (vestingDuration == 0) revert INVALID_VESTING_DURATION();
        if (cliffLength > vestingDuration) revert INVALID_VESTING_CLIFF();
        if (recipient == address(0)) revert INVALID_RECIPIENT();
        if (amount == 0) revert INVALID_AMOUNT();

```

## Impact
The recipient can directly claim all assets.

## Code Snippet
https://github.com/sherlock-audit/2024-01-rio-vesting-escrow/blob/main/rio-vesting-escrow/src/VestingEscrowFactory.sol#L55
## Tool used

Manual Review

## Recommendation
Validate the vesting start time and end time.