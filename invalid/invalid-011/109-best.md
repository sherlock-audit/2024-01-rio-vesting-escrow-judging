Joyous Mango Bear

medium

# Missing Zero-Address Checks in `VestingEsrcowFactory.sol:updateVotingAdaptor` and `VestingEsrcowFactory.sol:changeManager` Functions,could lead to unintended consequences.

## Summary
The first function `VestingEsrcowFactory.sol:updateVotingAdaptor`, allows the owner to update the votingAdaptor address. The second function, `VestingEsrcowFactory.sol:changeManager`, enables the owner to change the manager address for the vesting contracts, if can affect the protocol if addresses are being set zero.

## Vulnerability Detail
The `updateVotingAdaptor()` and `changeManger()` function does not perform a zero address check on the new votingAdaptor address and manager address.The absence of a zero address check allows the owner to set the manager address to zero by miss, which may have unintended consequences or security implications

## Impact
If the`updateVotingAdaptor()` and `changeManger()` are set to zero address by miss, the main role of manger which is to revoke the unvested tokens from the contract `VestingEscrow.sol:revokeunvested()` if by some chance owner is unable to do that another role who have that power is manager if that address is being set to zero then, it is immpossible to use revoke functionality inside contract.
voting adaptor allows the Recipient to vote if adaptor address is set to zero they cannot vote.

## Code Snippet
https://github.com/sherlock-audit/2024-01-rio-vesting-escrow/blob/main/rio-vesting-escrow/src/VestingEscrowFactory.sol#L97
```javascript
  function updateVotingAdaptor(address _votingAdaptor) external onlyOwner {
       votingAdaptor = _votingAdaptor;
       emit VotingAdaptorUpgraded(_votingAdaptor);
   }

```
https://github.com/sherlock-audit/2024-01-rio-vesting-escrow/blob/main/rio-vesting-escrow/src/VestingEscrowFactory.sol#L103
```javascript
  function changeManager(address _manager) external onlyOwner {
       manager = _manager;
       emit ManagerChanged(_manager);
   }
```

## Tool used

Manual Review

## Recommendation
Implementing zero address checks will prevent potentially harmful scenarios where zero addresses are accepted for critical parameters

```diff
function updateVotingAdaptor(address _votingAdaptor) external onlyOwner {
+    require(_votingAdaptor != address(0),"Revert Reason: Address zero not allowed for voting adaptor");
       votingAdaptor = _votingAdaptor;
       emit VotingAdaptorUpgraded(_votingAdaptor);
   }

  function changeManager(address _manager) external onlyOwner {
+      require(_manager != address(0),"Revert Reason: Address zero not allowed for manager");
       manager = _manager;
       emit ManagerChanged(_manager);
  }
```
