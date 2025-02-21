# Guessing_game
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

contract NumberGuessingGame {
    address public owner;
    uint256 public prizeAmount;
    uint256 private secretNumber;

    // Event to notify when someone wins
    event Winner(address indexed player, uint256 prize);

    // Ensuring only the owner can call certain functions
    modifier onlyOwner() {
        require(msg.sender == owner, "Only the owner can call this");
        _;
    }

    // Setting the owner of the contract and initializing some variables
    function initializeGame() public {
        owner = msg.sender;
        prizeAmount = 1 ether;  // Prize amount for correct guesses
        secretNumber = uint256(blockhash(block.number - 1)) % 100; // Random number based on block hash
    }

    // The main game function - guess the number
    function guessNumber() external payable {
        require(msg.value == 0.01 ether, "You must send exactly 0.01 ETH to play");

        // The player's guess is generated automatically based on their address and block timestamp
        uint256 playerGuess = uint256(keccak256(abi.encodePacked(msg.sender, block.timestamp))) % 100;

        // Check if the guess matches the secret number
        if (playerGuess == secretNumber) {
            // Player wins! Transfer the prize
            payable(msg.sender).transfer(prizeAmount);
            emit Winner(msg.sender, prizeAmount);

            // Generate a new secret number for the next round
            secretNumber = uint256(blockhash(block.number - 1)) % 100;
        }
    }

    // Function to withdraw the contract balance (only the owner can do this)
    function withdrawBalance() external onlyOwner {
        payable(owner).transfer(address(this).balance);
    }
}
