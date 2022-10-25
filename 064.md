ctf_sec

medium

# chainId is missing in signature schema in Factory.sol so cross-chain signature reuse / replay is possible

## Summary

the chainId is missing the signature schema in Factory.sol so the signature can be reused cross-chain

## Vulnerability Detail

the signature can be reused cross-chain to execute transactions in Factory.sol

because the signature schema is not using chainId

```solidity
signedOnly(
    abi.encodePacked(msg.sender, templateName, initdata),
    signature
)
```

the signature data only includes msg.sender, templateName, initData, not nonce.

I add this because in the doc and in the readme, the project quote that the project will be multichain

https://github.com/sherlock-audit/2022-10-nftport/blob/main/evm-minting-master/Readme.md#fees-and-fee-management

>How to authenticate the no-service-fee versions?
can't really pre-register NFTPort API wallets in the factory - would be expensive due to the number of wallets we have (one per API key), multiple supported chains etc

## Impact

the current signature can be reused to execute transactions cross-chain.

## Code Snippet

https://github.com/sherlock-audit/2022-10-nftport/blob/main/evm-minting-master/contracts/Factory.sol#L163-L177

https://github.com/sherlock-audit/2022-10-nftport/blob/main/evm-minting-master/contracts/Factory.sol#L537-L543

## Tool used

Manual Review

## Recommendation

```solidity
function getChainID() external view returns (uint256) {
    uint256 id;
    assembly {
        id := chainid()
    }
    return id;
}
```

We recommend the project add chainId in the signature schema and increment the nonce to make sure the signature cannot be reused.
