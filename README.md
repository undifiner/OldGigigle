// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "@openzeppelin/contracts/token/ERC721/ERC721.sol";
import "@openzeppelin/contracts/access/Ownable.sol";

contract OldGigigle is ERC721, Ownable {
    uint256 public constant TOTAL_SUPPLY = 4;//total supply of the project
    uint256 public nextTokenId = 1;
    mapping(address => bool) private _whitelist;//the mapping checks if the wallet is in the wl
    address payable public setWithdrawalAddress;//the address of the  community wallet
    uint256 public whitelistEndTime;

    constructor() ERC721("OldGigigle", "OGG") {
        _whitelist[0x1234567890123456789012345678901234567890] = true;//or    _whitelist[0x1234567890123456789012345678901234567890,0x1234567890123456789012345678901234567890] = true;
        _whitelist[0x0987654321098765432109876543210987654321] = true;
        withdrawalAddress = payable(0x1234567890123456789012345678901234567890);
//withdrawalAddress1 = payable(0x1234567890123456789012345678901234567890);
//withdrawalAddress2 = payable(0x0987654321098765432109876543210987654321);
    }

    function mintFree() public {
        if (block.timestamp < whitelistEndTime) {
            require(_whitelist[msg.sender], "Address not whitelisted");
    } else if (block.timestamp < publicMintStartTime) {
            require(false, "Minting period has not started yet");
    } else if (block.timestamp >= publicMintEndTime) {
            require(false, "Minting period has ended");
    } else if (nextTokenId > TOTAL_SUPPLY) {
            require(false, "All tokens have been minted");
    } else {
            _safeMint(msg.sender, nextTokenId);
            nextTokenId++;

            // Check if all tokens have been minted and close minting if they have
        if (nextTokenId > TOTAL_SUPPLY) {
            publicMintEndTime = block.timestamp;
        }
    }
}

// function mintFree() public {
//  require(_whitelist[msg.sender] || block.timestamp >= publicMintStartTime, "Address not whitelisted and minting period has not started yet");
//   require(block.timestamp < publicMintEndTime, "Minting period has ended");
//    require(nextTokenId <= TOTAL_SUPPLY, "All tokens have been minted");

//   _safeMint(msg.sender, nextTokenId);
//  nextTokenId++;
//}
// The code bellow allows the minter to mint more than one to a price 0.003 eth
function batchMint(uint256 numTokens) public payable {
    require(block.timestamp >= publicMintStartTime, "Minting period has not started yet");
    require(block.timestamp < publicMintEndTime, "Minting period has ended");
    require(nextTokenId + numTokens <= TOTAL_SUPPLY + 1, "Not enough tokens left to mint");
    require(numTokens <= 5, "Exceeded maximum batch size");

    uint256 totalPrice = 0.003 ether * numTokens;
    require(msg.value >= totalPrice, "Insufficient funds");

    for (uint256 i = 0; i < numTokens; i++) {
        _safeMint(msg.sender, nextTokenId);
        nextTokenId++;
    }

    if (nextTokenId > TOTAL_SUPPLY) {
        publicMintEndTime = block.timestamp;
    }

    // refund any excess funds to the sender
    if (msg.value > totalPrice) {
        payable(msg.sender).transfer(msg.value - totalPrice);
    }
}
    function setBaseURI(string memory baseURI) public onlyOwner {
        _setBaseURI(baseURI);
    }

    function setTokenURI(uint256 tokenId, string memory tokenURI) public onlyOwner {
        _setTokenURI(tokenId, tokenURI);
    }

    function addToWhitelist(address addr) public onlyOwner {
        _whitelist[addr] = true;
         // set whitelist end time to 2 hours from now
         whitelistEndTime = block.timestamp + 2 hours; 
    }

    function removeFromWhitelist(address addr) public onlyOwner {
        _whitelist[addr] = false;
    }

    function isWhitelisted(address addr) public view returns (bool) {
        return _whitelist[addr];
    }

    function withdraw() public onlyOwner {
        payable(withdrawalAddress).transfer(address(this).balance);
    }

    function setWithdrawalAddress(address payable addr) public onlyOwner {
        withdrawalAddress = addr;
    }
//THIS CODE IS USED ONLY IF I WANT TO HAVE 2 WALLETS TO SPLIT THE ETH GENERATED FROM MINT
//function withdraw() public onlyOwner {
// uint256 balance = address(this).balance;
// uint256 amount1 = balance / 2; // split the balance equally between the two addresses
// uint256 amount2 = balance - amount1;
// payable(withdrawalAddress1).transfer(amount1);
// payable(withdrawalAddress2).transfer(amount2);
}

