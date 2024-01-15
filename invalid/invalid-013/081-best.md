Massive Porcelain Albatross

medium

# Unauthorized Asset Recovery

## Summary

The  functions `recoverERC20` and `recoverEther` from `VestingEscrow` contract return assets to the `recipient() address` potentially allowing recovery by a non-trusted role.

## Vulnerability Detail

The recipient address should only be authorized to receive the vested tokens in the contract and it represents the user role in the protocol. For any other emergency recoveries a trusted role in the protocol should be used, such as, the owner. Functions `recoverERC20` and `recoverEther` from `VestingEscrow` contract should transfer tokens or eth to the `owner`.

## Impact

This vulnerability could lead to a situation where tokens or Ether are recovered by an unintended recipient. Moreover, recipient may not be able to receive eth and it is immutable or in certain tokens can be whitelisted. So `owner` still is a better choice in terms of emergency as it can be changed because it is taken from the factory contract.

## Code Snippet

[VestingEscrow.sol#L209](https://github.com/sherlock-audit/2024-01-rio-vesting-escrow/blob/main/rio-vesting-escrow/src/VestingEscrow.sol#L209)
[VestingEscrow.sol#L218](https://github.com/sherlock-audit/2024-01-rio-vesting-escrow/blob/main/rio-vesting-escrow/src/VestingEscrow.sol#L218)

## Tool used

Manual Review

## Recommendation

Modify the `recoverERC20` and `recoverEther` functions to transfer recovered tokens and Ether to a more trusted address, such as the `owner` of the `VestingEscrow` contract that is taken from the `VestingEscrowFactory`. This ensures that only authorized entities have the ability to recover assets in emergency situations
