GimelSec

high

# Attackers can bypass `tokensPerMint` and mint lots of tokens in a transaction

## Summary

Attackers can bypass `tokensPerMint`, allowing them to mint over `tokensPerMint` per transaction.

## Vulnerability Detail

NFTCollection has `tokensPerMint` that enforces the maximum number of tokens the user can mint per transaction. But attackers can bypass `tokensPerMint` by reentrancy attack to call `mint()` or `presaleMint()` multiple times in a transaction.

For example, we assume that `tokensPerMint` is `5` and we have a large amount of `availableSupply()`:
1. Alice (attacker) first creates an `AttackerContract` to call `mint(5)`, then [L318](https://github.com/sherlock-audit/2022-10-nftport/blob/main/evm-minting-master/contracts/templates/NFTCollection.sol#L318) will check that `amount <= _deploymentConfig.tokensPerMint` and call `_safeMint()` on L321.
2. But `_safeMint()` has a [callback](https://github.com/chiru-labs/ERC721A/blob/v3.2.0/contracts/ERC721A.sol#L392) that it will call `onERC721Received()` if `to` address is a contract.
3. Now `_safeMint()` calls `onERC721Received()` on the `AttackerContract`, and the `AttackerContract` re-enter `mint(5)` again. `mint()` function will pass the check of `tokensPerMint` on L318 again. Finally, Alice will mint 5+5 tokens in a transaction.

## Impact

Attackers can re-enter `mint()` function again and again to bypass the check of `tokensPerMint`, and mint lots of tokens in a transaction.

## Code Snippet

https://github.com/sherlock-audit/2022-10-nftport/blob/main/evm-minting-master/contracts/templates/NFTCollection.sol#L317-L322

## Tool used

Manual Review

## Recommendation

If the protocol wants to check `to` address, keep `_safeMint()` and add [ReentrancyGuard.sol](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/security/ReentrancyGuard.sol) on `mint()` and `presaleMint()` to prevent reentrancy attack.

Or just replace `_safeMint()` with `_mint()`.
