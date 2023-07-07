I found H3, M12 and QGA for 40 hours.

This audit for Angle protocol consists of two main parts, transmitter, and merkl. The test and the documentation were good and especially the whitepaper was great. I started with the transmuter because it contains vast mathematical logic for the stablecoin trading actions. The transmuter uses the Diamond Proxy pattern and I was familiar with the pattern from the ZkSynk contest.

The main functionalities of the transmuter are redemption and mint/burn. The Redeemer contract implements redemption and it returns all of the backing collaterals while redeeming. During redemption, Redeemer gets the values of collaterals from the oracle and uses those values so this protocol will be robust during black swan events. Mint/burn are handled in the Swapper contract. The main contribution of Swapper is setting fees so they can stabilize the total collateral ratio by manipulating of fee price of mint and burn. We can also specify exact input and output amounts and huge mathematical perspectives are used in whitepaper and implementation. I found a mistake on the whitepaper, but the implementation was good, fortunately.

I moved to the merkl module and it handles the distribution of rewards. Those distribution logics are interesting and I enjoyed finding several vulnerabilities there. I feel Distribution contracts are more centralized because the governor and guardians can affect the reward distribution a lot. There were some instances of array copying and if we resize instead of copy, some gas will be saved.


### Time spent:
40 hours