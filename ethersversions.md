# Ethers version

Using hardhat you could encounter different version of the ether package.  
Currently we are on ether v6 but you could find tutorials and examples on internet regarding ether v4 or v5.  
Be sure to use the correct syntax for the version of ether you are using.
I only encountered issues with hardhat test at that point (using the BigNumber class that change to BigInt and simplify things).

## How to know which ethers version i am running

You can write a simple test in your project to check ethers version :
```js
const { ethers } = require('hardhat')

describe('EthersVersion', function() {
  it('Should write ethers version', async () => {
    console.log(ethers.version)
  })
})
```

Then run it using hardhat :
```sh
npx hardhat test ./test/<name_of_my_file_test>.js
```

You should see the result like so :
```sh 
  EthersVersion
6.11.1
    ✔ Should write ethers version (62ms)
```
