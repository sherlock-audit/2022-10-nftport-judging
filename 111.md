sorrynotsorry

medium

# No fee validation

## Summary
No fee validation is made during setting the fees which can cause a loss of funds for the users.
## Vulnerability Detail
Factory contract has  `setDeploymentFee` and `setCallFee` functions to set the fees under admin privilege.
However, there is no validation carried out while setting those. The fees should be in between boundaries to prevent human error and loss of user funds due to this. E.g., the intention is to set the new fee to 10% but it ends up setting it to 100%. 

Recently Optimism Governance tokens' yearly inflation rate is mistakenly set to 20% instead of 2%.
[Reference](https://tokeninsight.com/en/news/optimism-op-inflation-rate-mistakenly-set-at-20-to-be-changed-back-to-2-today)

## Impact
Loss of user funds
## Code Snippet
```solidity
    function setDeploymentFee(uint256 newFee) external onlyRole(ADMIN_ROLE) {
        deploymentFee = newFee;
    }
```
[Permalink](https://github.com/sherlock-audit/2022-10-nftport/blob/main/evm-minting-master/contracts/Factory.sol#L310-L312)
```solidity
    function setCallFee(uint256 newFee) external onlyRole(ADMIN_ROLE) {
        callFee = newFee;
    }
```
[Permalink](https://github.com/sherlock-audit/2022-10-nftport/blob/main/evm-minting-master/contracts/Factory.sol#L319-L321)
## Tool used

Manual Review

## Recommendation
Implement boundaries for those.