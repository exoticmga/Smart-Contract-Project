# Smart-Contract-Project
This is my first smart contract project build with hardhat.
waveportal.sol
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.0;

import "hardhat/console.sol";

contract WavePortal {
    uint256 totalWaves;

/*
     * We will be using this below to help generate a random number
     */
    uint256 private seed;

    /*
     * A little magic, Google what events are in Solidity!
     */
    event NewWave(address indexed from, uint256 timestamp, string message);

    /*
     * I created a struct here named Wave.
     * A struct is basically a custom datatype where we can customize what we want to hold inside it.
     */
    struct Wave {
        address waver; // The address of the user who waved.
        string message; // The message the user sent.
        uint256 timestamp; // The timestamp when the user waved.
    }

    /*
     * I declare a variable waves that lets me store an array of structs.
     * This is what lets me hold all the waves anyone ever sends to me!
     */
    Wave[] waves;

/*
     * This is an address => uint mapping, meaning I can associate an address with a number!
     * In this case, I'll be storing the address with the last time the user waved at us.
     */
    mapping(address => uint256) public lastWavedAt;

    constructor() payable {
        console.log("I AM SMART CONTRACT. POG.");
  console.log("We have been constructed!");

/*
         * Set the initial seed
         */
        seed = (block.timestamp + block.difficulty) % 100;

    }


     /*
     * You'll notice I changedthe wave function a little here as well and
     * now it requires a string called _message. This is the message our user
     * sends us from the frontend!
     */
    function wave(string memory _message) public {

/*
         * We need to make sure the current timestamp is at least 15-minutes bigger than the last timestamp we stored
         */
        require
            (lastWavedAt[msg.sender] + 30 seconds < block.timestamp, 
            "Must wait 30 seconds before waving again.");
        /*
         * Update the current timestamp we have for the user
         */
        lastWavedAt[msg.sender] = block.timestamp;

        totalWaves += 1;
            console.log("%s has waved!", msg.sender);
        console.log("%s waved w/ message %s", msg.sender, _message);

       /*
         * This is where I actually store the wave data in the array.
         */
        waves.push(Wave(msg.sender, _message, block.timestamp));

/*
         * Generate a new seed for the next user that sends a wave
         */
        seed = (block.difficulty + block.timestamp + seed) % 100;

        console.log("Random # generated: %d", seed);

/*
         * Give a 50% chance that the user wins the prize.
         */
        if (seed <= 50) {
            console.log("%s won!", msg.sender);

        /*
         * I added some fanciness here, Google it and try to figure out what it is!
         * Let me know what you learn in #general-chill-chat
         */
        emit NewWave(msg.sender, block.timestamp, _message);
        
        uint256 prizeAmount = 0.0001 ether;
    require(
        prizeAmount <= address(this).balance,
        "Trying to withdraw more money than the contract has."
    );
    (bool success, ) = (msg.sender).call{value: prizeAmount}("");
    require(success, "Failed to withdraw money from contract.");

        }
  
    emit NewWave(msg.sender, block.timestamp, _message);
    }

     /*
     * I added a function getAllWaves which will return the struct array, waves, to us.
     * This will make it easy to retrieve the waves from our website!
     */
    function getAllWaves() public view returns (Wave[] memory) {
        return waves;
    }
    function getTotalWaves() public view returns (uint256) {
         // Optional: Add this line if you want to see the contract print the value!
        // We'll also print it over in run.js as well.
        console.log("We have %d total waves!", totalWaves);
        return totalWaves;
    }
}

run.js
const main = async () => {
  const [owner, randomPerson] = await hre.ethers.getSigners();
  const waveContractFactory = await hre.ethers.getContractFactory("WavePortal");
  const waveContract = await waveContractFactory.deploy({
      value: hre.ethers.utils.parseEther("0.1"),
  });

  await waveContract.deployed();
  console.log("Contract addy:", waveContract.address);
  /*
   * Get Contract balance
   */
  let contractBalance = await hre.ethers.provider.getBalance(
    waveContract.address
  );
  console.log(
    "Contract balance:",
    hre.ethers.utils.formatEther(contractBalance)
  );

/*
   * Let's try two waves now
   */
const waveTxn = await waveContract.wave("This is wave #1");
await waveTxn.wait();

const waveTxn2 = await waveContract.wave("This is wave #2");
await waveTxn2.wait();

contractBalance = await hre.ethers.provider.getBalance(waveContract.address);
console.log(
  "Contract balance:",
  hre.ethers.utils.formatEther(contractBalance)
);

  /*
   * Send Wave
   */
  console.log("Contract deployed to:", waveContract.address);
  console.log("Contract deployed by:", owner.address);

  let waveCount;
  waveCount = await waveContract.getTotalWaves();
  console.log(waveCount.toNumber());
  
  /**
  * Let's send a few waves!
  */
 
  /*
   * Get Contract balance to see what happened!
   */
  contractBalance = await hre.ethers.provider.getBalance(waveContract.address);
  console.log(
    "Contract balance:",
    hre.ethers.utils.formatEther(contractBalance)
  );
  let allWaves = await waveContract.getAllWaves();
  console.log(allWaves);
  waveCount = await waveContract.getTotalWaves(); 
 };

 const runMain = async () => {
     try {
         await main();
         process.exit(0); // exit Node process without error
     }   catch (error) {
         console.log(error);
         process.exit(1);
     }
};

runMain ();

deploy.js
const main = async () => {
    const [deployer] = await hre.ethers.getSigners();
    const accountBalance = await deployer.getBalance();

    console.log("Deploying contracts with account: ", deployer.address);
    console.log("Account balance: ", accountBalance.toString());

    const waveContractFactory = await hre.ethers.getContractFactory("WavePortal");
    const waveContract = await waveContractFactory.deploy({
        value: hre.ethers.utils.parseEther("0.001"),
    });
    
    await waveContract.deployed();

    console.log("WavePortal address: ", waveContract.address);
};

const runMain = async () => {
    try {
        await main ();
        process.exit(0);
    }   catch (error) {
        console.log(error);
        process.exit(1);
    }
};

runMain();
