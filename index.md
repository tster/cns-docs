# Canto Namespace Protocol

Canto Namespace Protocol (CNS) is a subprotocol for the Canto Identity Protocol that enables users to mint names with tiles (characters) that are contained in trays.

## Overview

CNS consists of three smart contracts:

* [`Namespace.sol`](https://github.com/mkt-market/canto-namespaces-protocol/blob/master/src/Namespace.sol)
* [`Tray.sol`](https://github.com/mkt-market/canto-namespaces-protocol/blob/master/src/Tray.sol)
* [`Utils.sol`](https://github.com/mkt-market/canto-namespaces-protocol/blob/master/src/Utils.sol)

### Namespace.sol

`Namespace.sol` is an ERC721 contract responsible for the minting of Namespace NFTs, which are unique on-chain display names. Each name publicly maps to its Namespace NFT tokenId, which can in turn be resolved to an address using the core Canto Identity Protocol.

### Tray.sol

`Tray.sol` is an ERC721 contract responsible for the minting of Tray NFTs. Trays consist of 7 tiles (characters) chosen randomly at the time of minting from a pool of 909 arbitrarily weighted unicode characters.

### Utils.sol

`Utils.sol` is a helper contract which contains methods to:

* convert characters to unicode bytes
* generate SVGs for an array of tiles
* generate pseudorandom numbers
* get character UTF-8 encoding

## Fusing Names

To mint a Namespace NFT, users call the `fuse` method on `Namespace.sol` to "fuse" one or more tiles from any number of Tray NFTs held in their wallet.

The method takes an array of `CharacterData` structs as its only input. Each `CharacterData` item identifies a specific tile in a Tray NFT and modifies its skin tone (for supported emojis). The order of items in the array determines the order of characters in the resulting name.

```solidity
    struct CharacterData {
        /// @notice ID of the Tray NFT
        uint256 trayID;
        /// @notice Offset of the tile within the tray. Valid values 0..TILES_PER_TRAY - 1
        uint8 tileOffset;
        /// @notice Emoji modifier for the skin tone. Can have values of 0 (yellow) and 1 - 5 (light to dark). Only supported by some emojis
        uint8 skinToneModifier;
    }
```

Upon fusing, the Tray NFTs corresponding to the fused tiles are burnt, regardless of how many tiles were used from each tray.

Names must have between 1 and 13 tiles. The cost to fuse a name is 2^(13 - numCharacters) $NOTE.

### Approvals

Users must approve transfer of Tray NFTs and spending of $NOTE by `Namespace.sol` on the respective token contracts before fusing a name.

## Prelaunch

Canto Namespace Protocol has a prelaunch phase during which the contract owner of `Tray.sol` can premint 1000 Tray NFTs for arbitrary distribution. These trays can only be fused during the prelaunch period, after which they become invalid and revert on `tokenURI` calls.

The prelaunch phase ends when the contract owner calls `endPrelaunchPhase`.