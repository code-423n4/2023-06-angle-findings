## 1. VARIABLE SHADOWING SHOULD BE AVOIDED

The `Swapper._buildPermitTransferPayload()` function is used to create the `payload` for the `permitTransferFrom` function to be executed. In the `_buildPermitTransferPayload` there is an input parameter named `amount`. The same input parameter can be seen in the `TokenPermissions` struct as one of it element names. Hence this is leading to variable `shadowing` which can be confusing to both developers and auditors in the future. Hence it is recommeneded to change the `_buildPermitTransferPayload` function's `amount` variable name to `amountIn` for ease of readability and understanding of the code.

https://github.com/AngleProtocol/angle-transmuter/blob/9707ee4ed3d221e02dcfcd2ebaa4b4d38d280936/contracts/transmuter/facets/Swapper.sol#L492
https://github.com/AngleProtocol/angle-transmuter/blob/9707ee4ed3d221e02dcfcd2ebaa4b4d38d280936/contracts/transmuter/facets/Swapper.sol#L506

## 2. IMPLEMENT COLLATERAL BALANCE CHECK IN THE `Swapper` CONTRACT

The `Swapper.quoteIn` and `Swapper.quoteOut` are `external` `view` functions used by the users to get a quote on thier trades before the trades are actually executed via `swap` functions. Here in the above two functions an internal function `Swapper._checkAmounts` is called. This function checks if the collateral is a managed one, whether the managing contract has enough collateral asset left in its contract for the trade. It reverts if it doesnt' as shown below:

        if (collatInfo.isManaged > 0 && LibManager.maxAvailable(collatInfo.managerData.config) < amountOut)
            revert InvalidSwap();

But the `Swapper._checkAmounts` does not check whether the `Swapper` contract has enough collateral left for the trade if the collateral is not a managed one. Hence if the collateral is not a managed one, the transaction will only revert during the `swap` execution thus wasting the `gas` spent by the trader.

Hence it is recommended to include the collateral balance check for the `Swapper` contract for the respective collateral inside the `Swapper._checkAmounts()` function. 

https://github.com/AngleProtocol/angle-transmuter/blob/9707ee4ed3d221e02dcfcd2ebaa4b4d38d280936/contracts/transmuter/facets/Swapper.sol#L450-L452

## 3. `uint32` SHOULD BE MODIFIED TO `uint64` TO AVOID `2106` ERROR

In the `DistributionParameters` struct the `epochStart` timestamp is declared as a `uint32`. The issue here is the `epochStart` will overflow after 2106-02-07 06:28:15 UTC. As a result the `DistributionCreator` smart contract will not be usable after that point in time.

Hence the functions such as `DistributionCreator._getRoundedEpoch` will revert due to the overflow of the `epochStart` variable.

Hence it is recommended to change the variable size of `epochStart` from `uint32` to `uint64`.

https://github.com/AngleProtocol/merkl-contracts/blob/1825925daef8b22d9d6c0a2bc7aab3309342e786/contracts/DistributionCreator.sol#L455-L457
https://github.com/AngleProtocol/merkl-contracts/blob/1825925daef8b22d9d6c0a2bc7aab3309342e786/contracts/DistributionCreator.sol#L284

## 4. USE EXPLICIT IMPORTS

When importing the external contracts or libraries to a particular smart contract it is recommended to import the required contract, library or structs etc... in those imported contracts explicitly using thier name for ease of code readability and understanding.

https://github.com/AngleProtocol/merkl-contracts/blob/1825925daef8b22d9d6c0a2bc7aab3309342e786/contracts/DistributionCreator.sol#L42-L49

## 5. INCLUDE BOTH OLD AND NEW VALUES WHEN EMITTING `events` FOR CRITICAL STATE CHANGES

In the `DistributionCreator.setUserFeeRebate` only the new `userFeeRebate` value is emitted inside the event as shown below:

        emit FeeRebateUpdated(user, userFeeRebate); 

 But it is recommended to include the old `userFeeRebate` value as well for future log references if the need arises.

https://github.com/AngleProtocol/merkl-contracts/blob/1825925daef8b22d9d6c0a2bc7aab3309342e786/contracts/DistributionCreator.sol#L394
https://github.com/AngleProtocol/merkl-contracts/blob/1825925daef8b22d9d6c0a2bc7aab3309342e786/contracts/DistributionCreator.sol#L388
https://github.com/AngleProtocol/merkl-contracts/blob/1825925daef8b22d9d6c0a2bc7aab3309342e786/contracts/DistributionCreator.sol#L434
https://github.com/AngleProtocol/merkl-contracts/blob/1825925daef8b22d9d6c0a2bc7aab3309342e786/contracts/DistributionCreator.sol#L442
https://github.com/AngleProtocol/merkl-contracts/blob/1825925daef8b22d9d6c0a2bc7aab3309342e786/contracts/DistributionCreator.sol#L381
https://github.com/AngleProtocol/merkl-contracts/blob/1825925daef8b22d9d6c0a2bc7aab3309342e786/contracts/Distributor.sol#L287
https://github.com/AngleProtocol/merkl-contracts/blob/1825925daef8b22d9d6c0a2bc7aab3309342e786/contracts/Distributor.sol#L294
https://github.com/AngleProtocol/merkl-contracts/blob/1825925daef8b22d9d6c0a2bc7aab3309342e786/contracts/Distributor.sol#L301
https://github.com/AngleProtocol/angle-transmuter/blob/9707ee4ed3d221e02dcfcd2ebaa4b4d38d280936/contracts/transmuter/facets/Redeemer.sol#L211
https://github.com/AngleProtocol/angle-transmuter/blob/9707ee4ed3d221e02dcfcd2ebaa4b4d38d280936/contracts/savings/Savings.sol#L230

## 6. DISCPREPENCY IN THE `if` STATEMENTS SHOULD BE FIXED

In the `DistributionCreator.setFees` function the `if` condition checks the value of the `_fees` as shown below:

        if (_fees >= BASE_9) revert InvalidParam();

In the `DistributionCreator.initialize()` function the `if` condition checks the value of the `_fees` as shown below:

        if (_fees > BASE_9) revert InvalidParam();

Hence as it is seen above one `if` clause uses `>` and other one uses `>=`. Hence there is a discrepency between the two implementation. Hence it is recommended to change this inside the `DistributionCreator.initialize()` as follows:

        if (_fees >= BASE_9) revert InvalidParam();

https://github.com/AngleProtocol/merkl-contracts/blob/1825925daef8b22d9d6c0a2bc7aab3309342e786/contracts/DistributionCreator.sol#L386
https://github.com/AngleProtocol/merkl-contracts/blob/1825925daef8b22d9d6c0a2bc7aab3309342e786/contracts/DistributionCreator.sol#L149

## 7. ARRAY LENGTHS OF THE TWO ARRAYS SHOULD BE CHECKED FOR EQUALIT

The `DistributionCreator.setRewardTokenMinAmounts()` function, is used to set the minimum reward token amount per epoch for each reward token. This function accepts two arrays as shown below:

        address[] calldata tokens,
        uint256[] calldata amounts

The length of the `tokens` array is taken as the upperlimit for the `for` loop to iterate over.

The issue here is, there is no check to verify that both `tokens` array and `amounts` array are of the same length. Hence if there is mismatch of array lengths the transaction will revert after consuming lot of gas, at end of the smaller array length.

Hence It is recommeneded to check the array lengths of the two arrays for `eqaulity` before iterating over the `for` loop.

https://github.com/AngleProtocol/merkl-contracts/blob/1825925daef8b22d9d6c0a2bc7aab3309342e786/contracts/DistributionCreator.sol#L416-L429

## 8. EMIT EVENTS FOR THE CRITICAL STATE CHANGES 

The `SavingsVest.setSurplusManager` changes the address of the `surplusManager` state variable. This is the address which manges the surplus profit share of the protocol. Hence this is a critical state change since this address governs funds owned by the protocol. Hence it is recommended to emit an event during the state change of `surplusManager` state variable.

https://github.com/AngleProtocol/angle-transmuter/blob/9707ee4ed3d221e02dcfcd2ebaa4b4d38d280936/contracts/savings/SavingsVest.sol#L191-L193

## 9. NO CHECK TO VERFIY `vestingPeriod` IS NOT INITIALIZED TO ZERO

In the `SavingsVest` contract the `vestingPeriod` is defined as the period in seconds over which locked profit is unlocked. In the `Natspec` comments for hte above varaible it is mentioned that `vestingPeriod` can not be `0` as shown below:

    /// @dev Cannot be 0 as it opens the system to sandwich attacks

But when the `vestingPeriod` state variable set in the `Savingsvest.setParams()` function, there is no conditional check to make sure that the `vestingPeriod` is not set to `0`. Because the `vestingPeriod` is directly set the truncated value of `params` as shown below:

        else if (what == "VP") vestingPeriod = uint32(param);

Hence it is recommended to add a conditional check to make sure tha `vestingPeriod != 0` when setting the state variable.

https://github.com/AngleProtocol/angle-transmuter/blob/9707ee4ed3d221e02dcfcd2ebaa4b4d38d280936/contracts/savings/SavingsVest.sol#L39
https://github.com/AngleProtocol/angle-transmuter/blob/9707ee4ed3d221e02dcfcd2ebaa4b4d38d280936/contracts/savings/SavingsVest.sol#L199

## 10. FEE ON TRANSFER TOKENS AS COLLATERAL COULD BREAK INTERNAL ACCOUNTING

In the `Swapper._swap()` internal function, the collateral asset is transfered to either the maanging strategy or the `Swapper` contract itself and pre-calculated `AgTokens` are minted to the `to` address.

But if the `Collateral` token is a fee on transfer token then the received collateral will be less than the amount used in the `Swapper.swapExactInput()` function calculate the `AgToken` `amountOut`.

This could break the entire accounting system of the protocol and revert the transaction when the user is burning or redeeming the `AgToken` for the collateral amount later. Since now the protocol does not have enough collateral stored in the smartcontract to transfer, since some collateral amount was spent as `transfer fee` initially.

Hence either fee on transfer tokens should not be used as collateral in this protocol or if they are used then the fee percentage should be taken into account when calculating the respective `AgToken` amount.

https://github.com/AngleProtocol/angle-transmuter/blob/9707ee4ed3d221e02dcfcd2ebaa4b4d38d280936/contracts/transmuter/facets/Swapper.sol#L199
https://github.com/AngleProtocol/angle-transmuter/blob/9707ee4ed3d221e02dcfcd2ebaa4b4d38d280936/contracts/transmuter/facets/Swapper.sol#L202

## 11. TYPOS IN THE NATSPEC COMMENTS

    // The `stablecoinsIssued` value need to be rounded up because it is then used as a `divizer` when computing
    // the `amount of stablecoins issued`
    Above should be corrected as follows:
    // The `stablecoinsIssued` value need to be rounded up because it is then used as a `divisor` when computing
    // the collatRatio

https://github.com/AngleProtocol/angle-transmuter/blob/9707ee4ed3d221e02dcfcd2ebaa4b4d38d280936/contracts/transmuter/libraries/LibGetters.sol#L82-83

    /// @notice Returns the list of all distributions that were or will be live between `epochStart` (included) and `epochEnd` (excluded)
    Above should be corrected as follows:
    /// @notice Returns the list of all distributions that were or will be live `at some point` between `epochStart` (included) and `epochEnd` (excluded)    

https://github.com/AngleProtocol/merkl-contracts/blob/1825925daef8b22d9d6c0a2bc7aab3309342e786/contracts/DistributionCreator.sol#L356

    /// @dev It is only possible to create a dispute "for" `disputePeriod` after each tree update
    Above word inside the "" should be corrected as follows:
    /// @dev It is only possible to create a dispute "within" `disputePeriod` after each tree update    

https://github.com/AngleProtocol/merkl-contracts/blob/1825925daef8b22d9d6c0a2bc7aab3309342e786/contracts/Distributor.sol#L233

    /// @notice Time `before` which a change in a tree becomes effective, in EPOCH_DURATION
    Above should be corrected as follows:
    /// @notice Time `after` which a change in a tree becomes effective, in EPOCH_DURATION

https://github.com/AngleProtocol/merkl-contracts/blob/1825925daef8b22d9d6c0a2bc7aab3309342e786/contracts/Distributor.sol#L90

    /// @notice Sets the dispute period `before` which a tree update becomes effective
    Above should be corrected as follows:
    /// @notice Sets the dispute period `after` which a tree update becomes effective    

https://github.com/AngleProtocol/merkl-contracts/blob/1825925daef8b22d9d6c0a2bc7aab3309342e786/contracts/Distributor.sol#L284   

    /// or burning `some` if it is not collateralized
    Above should be corrected as follows:
    /// or burning `some,` if it is not collateralized    

https://github.com/AngleProtocol/angle-transmuter/blob/9707ee4ed3d221e02dcfcd2ebaa4b4d38d280936/contracts/savings/SavingsVest.sol#L102
