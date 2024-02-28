# Testing a smart contract

Here i will give some example where I explain in code inline what we are doing during a test

## Lending contract

Supposing we have this Lending smart contract : ./contracts/Lending.sol

```solidity 
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Lending {
  uint256 public availableFunds;
  address public depositor;
  uint256 public interest = 10;
  uint256 public repayTime;
  uint256 public repayed;
  uint256 public loanStartTime;
  uint256 public borrowed;
  bool public isLoanActive;

  constructor(uint256 _repayTime, uint256 _interest) {
    repayTime = _repayTime;
    interest = _interest;
    depositor = msg.sender;
    deposit();
  }
  
  // Permit to the lender to deposit some eth
  function deposit() public payable {
    require(msg.sender == depositor, 'You must be the depositor');
    availableFunds = availableFunds + msg.value;
  }

  // Permit to a borrower to borrow some eth
  function borrow(uint256 _amount) public {
    require(_amount > 0, 'Must borrow something');
    require(availableFunds >= _amount, 'No funds available');
    require(block.timestamp > loanStartTime + repayTime, 'Loan expired must be repayed first');

    if (borrowed == 0) {
      loanStartTime = block.timestamp;
    }
    availableFunds = availableFunds - _amount;
    borrowed = borrowed + _amount;
    isLoanActive = true;
    payable(msg.sender).transfer(_amount);
  }

  // Permit to a borrower to repay some eth he borrowed
  function repay() public payable {
    require(isLoanActive, 'Must be an active loan');
    uint256 amountToRepay = borrowed + (borrowed * interest / 100);
    uint256 leftToPay = amountToRepay - repayed;
    uint256 exceeding = 0;

    if (msg.value > leftToPay) {
      exceeding = msg.value - leftToPay;
      isLoanActive = false;
    } else if (msg.value == leftToPay) {
      isLoanActive = false;
    } else {
      repayed = repayed + msg.value;
    }

    payable(depositor).transfer(msg.value - exceeding);
    if (exceeding > 0) {
      payable(msg.sender).transfer(exceeding);
    }

    // Reset everything
    if (!isLoanActive) {
      borrowed = 0;
      availableFunds = 0;
      repayed = 0;
      loanStartTime = 0;
    }
  }
}
```

You can test it like so using javascript and hardhat : ./test/Lending.js

```js 
const { expect } = require('chai')
const { ethers } = require('hardhat')

// This variable will contain the Lending contract
// it is declared here to be in a higher scope and be
// available in all underlying test
let lending = null
describe('Lending', function() {
    
  // This method called "beforeEach" will be executed before every test
  // it will deploy the Lending smart contract to the test blockchain
  beforeEach(async () => {
    const Lending = await ethers.getContractFactory('Lending')
    lending = await Lending.deploy(30 * 24 * 60 * 60, 10) // 30 days in seconds
  })
  
  // This test will use every method and verify that they have the correct behavior :
  //   - the lender will deposit some eth
  //   - the borrower will borrow some eth
  //   - the borrower will repay some eth 
  it('Should repay a loan partially', async () => {
    // ethers.getSigners will give first the creator of the contract.
    // (the one who deployed it to the blockchain).
    // Then it will give address of other people 
    // on the blockchain (fake people here).
    const [lender, borrower] = await ethers.getSigners()

    // Here we are depositing one eth to the contract
    const oneEth = BigInt('1000000000000000000')
    await lending.deposit({ value: oneEth })
    expect(await lending.availableFunds()).to.eq(oneEth)

    // Here we are checking the original balance of the borrower
    const balance1 = await ethers.provider.getBalance(borrower.address)
    // This will change the person connected to the contract.
    // Originally, if we don't call the connect method,
    // it is the first address returned by ethers.getSigners().
    lending = lending.connect(borrower)
    // Then we borrow 1/2 eth from the contract
    const tx = await lending.borrow(oneEth / 2n)
    // tx represent the borrow transaction.
    // Here we are waiting for it to finish.
    const receipt = await tx.wait()
    // The receipt contains informations about finished transaction.
    // You can see what's inside by logging it to the console : console.log(receipt).
    const totalGasPrice = receipt.gasPrice * receipt.gasUsed
    // now we get the new balance from the borrower to check.
    // he now possess 1/2 of eth minus the gas cost of the transaction
    const balance2 = await ethers.provider.getBalance(borrower.address)
    expect(await lending.availableFunds()).to.eq(oneEth / 2n)
    expect(balance2).to.eq(balance1 + (oneEth / 2n) - totalGasPrice)

    // we get the current lender balance to verify the repay method
    const balance3 = await ethers.provider.getBalance(lender.address)
    // we run the repay transaction for 1/10 eth
    const tx_repay = await lending.repay({ value: oneEth / 10n })
    // the tx_repay represent this pending transaction so we, again, are 
    // waiting on it to finish
    const receipt_repay = await tx_repay.wait()
    // again we calculate the total gas cost of the transaction for verification
    const totalGasPrice_repay = receipt_repay.gasPrice * receipt_repay.gasUsed
    // then we get the new lender balance to check if he now possess 1/10 eth more
    const balance4 = await ethers.provider.getBalance(lender.address)
    expect(balance4).to.eq(balance3 + (oneEth / 10n))
    // then we get the new borrower balance to check he now have 1/10 eth less 
    // and paid the gas cost of the transaction 
    const balance5 = await ethers.provider.getBalance(borrower.address)
    expect(balance5).to.eq(balance2 - (oneEth / 10n) - totalGasPrice_repay)
  })
})
```
