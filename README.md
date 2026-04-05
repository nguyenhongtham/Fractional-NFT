# Fractional-NFT
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.26;

import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
import "@openzeppelin/contracts/token/ERC721/IERC721.sol";
import "@openzeppelin/contracts/access/Ownable.sol";

contract FractionalNFT is ERC20, Ownable {
    IERC721 public originalNFT;
    uint256 public tokenId;
    bool public fractionalized = false;

    constructor(address _nft, uint256 _tokenId, string memory name, string memory symbol)
        ERC20(name, symbol) {
        originalNFT = IERC721(_nft);
        tokenId = _tokenId;
    }

    function fractionalize(uint256 shares) external onlyOwner {
        require(!fractionalized, "Already done");
        originalNFT.transferFrom(msg.sender, address(this), tokenId);
        _mint(msg.sender, shares);
        fractionalized = true;
    }

    function redeem(uint256 amount) external {
        require(balanceOf(msg.sender) >= amount, "Not enough shares");
        _burn(msg.sender, amount);
        if (totalSupply() == 0) {
            originalNFT.transferFrom(address(this), msg.sender, tokenId);
        }
    }
}
