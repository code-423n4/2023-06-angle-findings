# [L-01] Missing indexed event field
## Targets
https://github.com/AngleProtocol/merkl-contracts/blob/1825925daef8b22d9d6c0a2bc7aab3309342e786/contracts/Distributor.sol#L111
## Impact
Querying and filtering `Claimed` events by username or token address will be difficult and in some cases impossible.
## Proof of Concept
The `Claimed` event doesn't have indexed fields ([Distributor.sol#L111](https://github.com/AngleProtocol/merkl-contracts/blob/1825925daef8b22d9d6c0a2bc7aab3309342e786/contracts/Distributor.sol#L111)):
```solidity
event Claimed(address user, address token, uint256 amount);
```
## Tools Used
Manual review
## Recommended Mitigation Steps
Consider adding indexed fields to the `Claimed` event. For example, indexing the `user` field will make it possible to find all tokens claimed by an address.



# [L-02] `answeredInRound` is deprecated
## Targets
https://github.com/AngleProtocol/angle-transmuter/blob/9707ee4ed3d221e02dcfcd2ebaa4b4d38d280936/contracts/transmuter/libraries/LibOracle.sol#L152-L153
## Impact
In some distant future, `answeredInRound` may start returning a stub value (e.g. 0), which will break the check for all cases.
## Proof of Concept
As per the [Chainlink documentation](https://docs.chain.link/data-feeds/historical-data#getrounddata-return-values), the `answeredInRound` variable has been deprecated:
> answeredInRound: Deprecated - Previously used when answers could take multiple rounds to be computed

Thus, it's no longer necessary to check that `roundId > answeredInRound`. In some distant future, `answeredInRound` may start returning a stub value (e.g. 0), which will break the check for all cases.
## Tools Used
Manual review
## Recommended Mitigation Steps
Consider removing the `roundId > answeredInRound` check from the `LibOracle.readChainlinkFeed` function.



# [L-03] Risky use of toggle functions
## Targets
https://github.com/AngleProtocol/angle-transmuter/blob/9707ee4ed3d221e02dcfcd2ebaa4b4d38d280936/contracts/transmuter/facets/SettersGuardian.sol#L16-L18
https://github.com/AngleProtocol/angle-transmuter/blob/9707ee4ed3d221e02dcfcd2ebaa4b4d38d280936/contracts/savings/Savings.sol#L220-L224
## Impact
The governor and the guardian may fail to pause the Transmuter or the Savings contract when pausing is needed.
## Proof of Concept
[SettersGuardian.togglePause](https://github.com/AngleProtocol/angle-transmuter/blob/9707ee4ed3d221e02dcfcd2ebaa4b4d38d280936/contracts/transmuter/facets/SettersGuardian.sol#L16-L18) and [Savings.togglePause](https://github.com/AngleProtocol/angle-transmuter/blob/9707ee4ed3d221e02dcfcd2ebaa4b4d38d280936/contracts/savings/Savings.sol#L220-L224) are functions used to pause the contracts, they can only be called by the governor or the guardian. These functions are error-prone since they depend on the current state of the blockchain. If the governor and the guardian independently trigger either of the functions at around the same time (e.g. an incident may force them to pause the contract ASAP), the contract won't actually be paused since the second call will unpause it.
## Tools Used
Manual review
## Recommended Mitigation Steps
For the two functions, consider replacing them with functions that explicitly set the paused state to either `0` or `1`.