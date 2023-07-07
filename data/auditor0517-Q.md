### [L-01] `setDisputePeriod()` should validate `disputePeriod > 0`
- https://github.com/AngleProtocol/merkl-contracts/blob/1825925daef8b22d9d6c0a2bc7aab3309342e786/contracts/Distributor.sol#L286

```solidity
    function setDisputePeriod(uint48 _disputePeriod) external onlyGovernorOrGuardian {
        disputePeriod = uint48(_disputePeriod); //@audit should check > 0
        emit DisputePeriodUpdated(_disputePeriod);
    }
```

If `disputePeriod == 0`, [_endOfDisputePeriod()](https://github.com/AngleProtocol/merkl-contracts/blob/1825925daef8b22d9d6c0a2bc7aab3309342e786/contracts/Distributor.sol#L321) will return the same time as `treeUpdate` when it's divisible by `_EPOCH_DURATION`. Then the merkle tree would be updated without any dispute period.

```solidity
    function _endOfDisputePeriod(uint48 treeUpdate) internal view returns (uint48) {
        return ((treeUpdate - 1) / _EPOCH_DURATION + 1 + disputePeriod) * (_EPOCH_DURATION);
    }
```

### [L-02] The merkle tree might be updated too soon by the governor.
- https://github.com/AngleProtocol/merkl-contracts/blob/1825925daef8b22d9d6c0a2bc7aab3309342e786/contracts/Distributor.sol#L220

```solidity
    function updateTree(MerkleTree calldata _tree) external {
        if (
            disputer != address(0) ||
            // A trusted address cannot update a tree right after a precedent tree update otherwise it can de facto
            // validate a tree which has not passed the dispute period
            ((canUpdateMerkleRoot[msg.sender] != 1 || block.timestamp < endOfDisputePeriod) &&
                !core.isGovernorOrGuardian(msg.sender))
        ) revert NotTrusted();
```

Users can dispute the merkle tree within `endOfDisputePeriod` but the tree might be confirmed anytime by the governor and it might break the fairness of the reward system.

### [L-03] Centralization risk
- https://github.com/AngleProtocol/merkl-contracts/blob/1825925daef8b22d9d6c0a2bc7aab3309342e786/contracts/Distributor.sol#L279

```solidity
    function recoverERC20(address tokenAddress, address to, uint256 amountToRecover) external onlyGovernorOrGuardian {
        IERC20(tokenAddress).safeTransfer(to, amountToRecover);
        emit Recovered(tokenAddress, to, amountToRecover);
    }
```

The rewards might be withdrawn by the governor.

### [L-04] Wrong comment
- https://github.com/AngleProtocol/angle-transmuter/blob/9707ee4ed3d221e02dcfcd2ebaa4b4d38d280936/contracts/transmuter/facets/Swapper.sol#L390

```solidity
//                      = (g(0)-1+sqrt[(1+g(0))**2+2M(f_{i+1}-g(0))/b_{i+1})]) @audit divide by 2
```

- https://github.com/AngleProtocol/angle-transmuter/blob/9707ee4ed3d221e02dcfcd2ebaa4b4d38d280936/contracts/transmuter/facets/Swapper.sol#L402

```solidity
//                      = (g(0)+1-sqrt[(1-g(0))**2-2M(f_{i+1}-g(0))/b_{i+1})]) @audit divide by 2
```

### [N-01] Variables need not be initialized to zero
- https://github.com/AngleProtocol/angle-transmuter/blob/9707ee4ed3d221e02dcfcd2ebaa4b4d38d280936/contracts/transmuter/facets/Redeemer.sol#L199
- https://github.com/AngleProtocol/angle-transmuter/blob/9707ee4ed3d221e02dcfcd2ebaa4b4d38d280936/contracts/transmuter/libraries/LibHelpers.sol#L23
- https://github.com/AngleProtocol/angle-transmuter/blob/9707ee4ed3d221e02dcfcd2ebaa4b4d38d280936/contracts/transmuter/libraries/LibSetters.sol#L233
