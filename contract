// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract MemeTokenPresale {
    address public owner;
    address public tokenAddress;
    uint256 public startTime;
    uint256 public endTime;
    uint256 public hardCap;
    uint256 public totalRaised;
    uint256 public tokenRate; // Tokens per ETH

    mapping(address => uint256) public contributions;

    bool public finalized;

    event ContributionReceived(address indexed contributor, uint256 amount);
    event TokensClaimed(address indexed claimer, uint256 amount);
    event RefundIssued(address indexed contributor, uint256 amount);
    event Finalized(uint256 totalRaised);

    modifier onlyOwner() {
        require(msg.sender == owner, "Only owner can call this function");
        _;
    }

    modifier duringPresale() {
        require(block.timestamp >= startTime && block.timestamp <= endTime, "Presale is not active");
        _;
    }

    modifier afterPresale() {
        require(block.timestamp > endTime, "Presale has not ended");
        _;
    }

    constructor(
        address _tokenAddress,
        uint256 _startTime,
        uint256 _endTime,
        uint256 _hardCap,
        uint256 _tokenRate
    ) {
        require(_startTime < _endTime, "Invalid time range");
        require(_hardCap > 0, "Hard cap must be greater than 0");
        require(_tokenRate > 0, "Token rate must be greater than 0");

        owner = msg.sender;
        tokenAddress = _tokenAddress;
        startTime = _startTime;
        endTime = _endTime;
        hardCap = _hardCap;
        tokenRate = _tokenRate;
    }

    function contribute() external payable duringPresale {
        require(totalRaised + msg.value <= hardCap, "Exceeds hard cap");
        require(msg.value > 0, "Contribution must be greater than 0");

        contributions[msg.sender] += msg.value;
        totalRaised += msg.value;

        emit ContributionReceived(msg.sender, msg.value);
    }

    function finalizePresale() external onlyOwner afterPresale {
        require(!finalized, "Presale already finalized");

        finalized = true;

        if (totalRaised > hardCap) {
            uint256 overflow = totalRaised - hardCap;

            for (address contributor = address(0); contributor != address(0); contributor++) {
                uint256 contribution = contributions[contributor];
                uint256 refundAmount = (contribution * overflow) / totalRaised;
                contributions[contributor] -= refundAmount;
                payable(contributor).transfer(refundAmount);

                emit RefundIssued(contributor, refundAmount);
            }
        }

        emit Finalized(totalRaised);
    }

    function claimTokens() external afterPresale {
        require(finalized, "Presale not finalized");

        uint256 contribution = contributions[msg.sender];
        require(contribution > 0, "No contribution found");

        uint256 tokenAmount = (contribution * tokenRate) / 1 ether;
        contributions[msg.sender] = 0;

        require(ERC20(tokenAddress).transfer(msg.sender, tokenAmount), "Token transfer failed");

        emit TokensClaimed(msg.sender, tokenAmount);
    }

    function withdrawFunds() external onlyOwner afterPresale {
        require(finalized, "Presale not finalized");
        payable(owner).transfer(address(this).balance);
    }

    function emergencyWithdraw() external onlyOwner {
        payable(owner).transfer(address(this).balance);
    }
}

interface ERC20 {
    function transfer(address to, uint256 value) external returns (bool);
}
