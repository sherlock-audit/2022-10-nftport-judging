obront

medium

# Presale mint remains open indefinitely

## Summary

Presale mints typically offer a lower price to a whitelist of user if they buy before the main sale. The purpose of this is to incentivize early purchases to start the momentum.

However, in the current `NFTCollection.sol` implementation, any whitelisted user is able to continue using the `presaleMint()` function to mint at a lower price after the public mint opens.

## Vulnerability Detail

When calling `presaleMint()`, the function calls out to `presaleActive()` to confirm that the purchase is permitted. This function simply checks if we are passed the presale mint time:

`return block.timestamp > _runtimeConfig.presaleMintStart;`

However, this logic does not consider that many creators do not want the premint to remain accessible after the public mint has begun.

## Impact

Users approved to premint have no incentive to actually mint in advance. Instead, they could wait until the public mint begins to see how successful the mint is, and use their premint whitelist spot for a discounted rate at any time.

Any creators who wanted to avoid this problem would need to both (a) understand that this was the case and (b) manually update the runtime configs immediately upon the transition to the public mint phase, which seems to be an undue burden to put on creators for a tool focused on creating simple infrastructure.

## Code Snippet

https://github.com/sherlock-audit/2022-10-nftport/blob/main/evm-minting-master/contracts/templates/NFTCollection.sol#L205-L210

## Tool used

Manual Review

## Recommendation

The cleanest way to do this is for the `presaleActive()` function to check that we are later than the presale mint start time, but also earlier than the public mint start time.

```solidity
return (
    block.timestamp > _runtimeConfig.presaleMintStart && 
    block.timestamp <= _runtimeConfig.publicMintStart
);
```

If you want to support users who want the presale to remain active through the public mint and those who don't, you could set this as a boolean as a part of the RuntimeConfigs, and incorporate that into the `presaleActive()` logic.