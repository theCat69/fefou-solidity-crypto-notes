# NG questplay quick tips

## Prerequisites

Read carefully in order to setup your env on the website.
Don't forget to look for hidden text on the right side where you can sometimes find a walkthrough.

Basically you will need : 
- Node.js
- Python 3 (a recent version)
- A code editor (VSCode is great)
- A solc compiler => I recommend to install [solc-select](https://github.com/crytic/solc-select)

## Basic solidity and hardhat knowledge

You should start by a basic understanding of solidity.
Start by setuping a basic solidity project with hardhat : [hardhat tuto](https://hardhat.org/tutorial/creating-a-new-hardhat-project)
If you look about set up env for solidity dev they will always tell you to use windows WSL if you are on windows.
I advise against it as it will make your setup really difficult to follow if you are a beginner.
I am personnaly using AtlasOS (basically windows10) and Ubuntu with no difference in my solidity dev workflow.

## Campaign sub directory and dependency not found issues

When you run 
```sh 
npm install
``` 
in a quest directory this will install in the ng-questplay node_modules and not in you current subdirectory.
If you need to have node_modules in the subdirectory for your quest project you can use the command:

```sh
cd <quest_folder>
npm install --prefix .
```

## hack it !

To help you in your quest there is tools you can install and use in your project :
- [slither](https://github.com/crytic/slither) : a static code analyser to find security issues in smart contracts. 
