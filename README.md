# Frax Ether Liquid Staking contest details
- $28,500 USDC main award pot
- $1,500 USDC gas optimization award pot
- Join [C4 Discord](https://discord.gg/code4rena) to register
- Submit findings [using the C4 form](https://code4rena.com/contests/2022-09-frax-ether-liquid-staking-contest/submit)
- [Read our guidelines for more details](https://docs.code4rena.com/roles/wardens)
- Starts September 22, 2022 20:00 UTC
- Ends September 25, 2022 20:00 UTC

# Frax Staked Ethereum
## Flowchart
![frxETH Flowchart](https://3191235985-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-MJQZW1mSg2O5N7HXHo0-1972196547%2Fuploads%2FhJQ4ici8EKE8INcS0AZ2%2Fflowchart.svg?alt=media&token=c9ac2edd-ccb3-430b-923d-220c6905947b)
## Overview Documentation
[https://docs.frax.finance/frax-ether/overview](https://docs.frax.finance/frax-ether/overview).


# Building and Testing
## Setup
1) Install [foundry](https://book.getfoundry.sh/getting-started/installation)
2) ```forge install```
3) ```git submodule update --init --recursive```
4) (Optional) Occasionally update / pull your submodules to keep them up to date. ```git submodule update --recursive --remote```
5) Create your own .env and copy SAMPLE.env into there. Sample mainnet validator deposit keys are in test/deposit_data-TESTS-MAINNET.json if you need more.
6) You don't need to add PRIVATE_KEY, ETHERSCAN_KEY, or FRXETH_OWNER if you are not actually deploying on live mainnet

Reccomended to make new dependencies play nicely with VSCode:
```forge remappings > remappings.txt```

Note: sometimes the alchemy URL can fail, preventing mainnet forking. If this happens change the RPC url in your .env to use another provider such as infura or alchemy

### Forge
Manually, forced
```forge build --force```

## Testing
### General / Helpful Notes
1) If you want to add more fuzz cycles, edit foundry.toml
2) Foundry [cheatcodes](https://book.getfoundry.sh/cheatcodes/)
### Forge
Most cases
```source .env && forge test -vv```

If you need to fork mainnet
```source .env && forge test --fork-url $MAINNET_RPC_URL -vv```

To generate gas reports
```source .env && forge test --gas-report --fork-url $MAINNET_RPC_URL -vv```

If you need to fork mainnet, single test contract
```source .env && forge test --fork-url $MAINNET_RPC_URL -vv --match-path ./test/frxETHMinter.t.sol```

Verbosely test a single contract while forking mainnet
or ```source .env && forge test --fork-url $MAINNET_RPC_URL -m test_frxETHMinter_submitAndDepositRegular -vvvvv``` for single test verbosity level 5

### Slither
1) Install [slither](https://github.com/crytic/slither#how-to-install)
2) Slither a single contract
```slither ./src/sfrxETH.sol --solc-remaps "openzeppelin-contracts=lib/openzeppelin-contracts ERC4626=lib/ERC4626/src solmate=lib/solmate/src"```

Note: If `forge remappings > remappings.txt` is run, `slither .` works without specifying the remaps


<!-- //////////////////////////////////////////////////////////////// -->
# Deployment & Environment Setup
## Deploy
1) Deploy frxETH.sol
2) Deploy sfrxETH.sol
3) Deploy frxETHMinter.sol
4) Add the frxETHMinter as a valid minter for frxETH
5) (Optional, depending on how you want to test) Add some validators to frxETHMinter

### Goerli
#### Single deploy
```forge create src/frxETH.sol:frxETH --private-key $PRIVATE_KEY --rpc-url $GOERLI_RPC_URL --verify --optimize --etherscan-api-key $ETHERSCAN_KEY --constructor-args $FRXETH_OWNER $TIMELOCK_ADDRESS```

#### Group deploy script
```forge script script/deployGoerli.s.sol:Deploy --rpc-url $GOERLI_RPC_URL --private-key $PRIVATE_KEY --broadcast --verify --etherscan-api-key $ETHERSCAN_KEY```

#### Etherscan Verification
Sometimes the deploy scripts above fail with Etherscan's verification API. In that case, use:
```forge flatten src/frxETHMinter.sol -o flattened.sol```
Then do
```sed -i '/SPDX-License-Identifier/d' ./flattened.sol && sed -i '/pragma solidity/d' ./flattened.sol && sed -i '1s/^/\/\/ SPDX-License-Identifier: GPL-2.0-or-later\npragma solidity >=0.8.0;\n\n/' flattened.sol```

## All-in-one command
All in one command to setup and test the repository using your own RPC url
`export FORK_URL='<your_alchemy_mainnet_url_goes_here>' && rm -Rf 2022-09-frax || true && git clone https://github.com/code-423n4/2022-09-frax.git && cd 2022-09-frax && git submodule update --init --recursive --remote && foundryup && forge install && forge build && forge remappings > remappings.txt && cp SAMPLE.env .env && source .env && FORGE_GAS_REPORT=true forge test --fork-url $FORK_URL -vv`

All in one command to setup and test the repository using the default Ankr RPC
`rm -Rf 2022-09-frax || true && git clone https://github.com/code-423n4/2022-09-frax.git && cd 2022-09-frax && git submodule update --init --recursive --remote && foundryup && forge install && forge build && forge remappings > remappings.txt && cp SAMPLE.env .env && source .env && FORGE_GAS_REPORT=true forge test --fork-url $MAINNET_RPC_URL -vv`

## Scope
### Files in scope
|File|[SLOC](#nowhere "(nSLOC, SLOC, Lines)")|[Coverage](#nowhere "(Lines hit / Total)")|
|:-|:-:|:-:|
|_Contracts (5)_|
|[src/frxETH.sol](https://github.com/code-423n4/2022-09-frax/blob/main/src/frxETH.sol)|[10](#nowhere "(nSLOC:10, SLOC:10, Lines:43)")|-|
|[src/sfrxETH.sol](https://github.com/code-423n4/2022-09-frax/blob/main/src/sfrxETH.sol)|[42](#nowhere "(nSLOC:26, SLOC:42, Lines:89)")|[88.89%](#nowhere "(Hit:8 / Total:9)")|
|[src/ERC20/ERC20PermitPermissionedMint.sol](https://github.com/code-423n4/2022-09-frax/blob/main/src/ERC20/ERC20PermitPermissionedMint.sol)|[68](#nowhere "(nSLOC:68, SLOC:68, Lines:107)")|[10.00%](#nowhere "(Hit:2 / Total:20)")|
|[src/frxETHMinter.sol](https://github.com/code-423n4/2022-09-frax/blob/main/src/frxETHMinter.sol) [💰](#nowhere "Payable Functions") [📤](#nowhere "Initiates ETH Value Transfer")|[119](#nowhere "(nSLOC:119, SLOC:119, Lines:213)")|[88.10%](#nowhere "(Hit:37 / Total:42)")|
|[src/OperatorRegistry.sol](https://github.com/code-423n4/2022-09-frax/blob/main/src/OperatorRegistry.sol)|[126](#nowhere "(nSLOC:105, SLOC:126, Lines:216)")|[80.43%](#nowhere "(Hit:37 / Total:46)")|
|_Abstracts (1)_|
|[lib/ERC4626/src/xERC4626.sol](https://github.com/corddry/ERC4626/blob/643cd044fac34bcbf64e1c3790a5126fec0dbec1/src/xERC4626.sol)|[48](#nowhere "(nSLOC:48, SLOC:48, Lines:98)")|-|
|Total (over 6 files):| [413](#nowhere "(nSLOC:376, SLOC:413, Lines:766)")| [71.79%](#nowhere "Hit:84 / Total:117")|

## External imports
* **ERC4626/xERC4626.sol**
  * [src/sfrxETH.sol](https://github.com/code-423n4/2022-09-frax/blob/main/src/sfrxETH.sol)
* **openzeppelin-contracts/contracts/security/ReentrancyGuard.sol**
  * [src/frxETHMinter.sol](https://github.com/code-423n4/2022-09-frax/blob/main/src/frxETHMinter.sol)
  * [src/sfrxETH.sol](https://github.com/code-423n4/2022-09-frax/blob/main/src/sfrxETH.sol)
* **openzeppelin-contracts/contracts/token/ERC20/ERC20.sol**
  * [src/ERC20/ERC20PermitPermissionedMint.sol](https://github.com/code-423n4/2022-09-frax/blob/main/src/ERC20/ERC20PermitPermissionedMint.sol)
* **openzeppelin-contracts/contracts/token/ERC20/extensions/draft-ERC20Permit.sol**
  * [src/ERC20/ERC20PermitPermissionedMint.sol](https://github.com/code-423n4/2022-09-frax/blob/main/src/ERC20/ERC20PermitPermissionedMint.sol)
* **openzeppelin-contracts/contracts/token/ERC20/extensions/ERC20Burnable.sol**
  * [src/ERC20/ERC20PermitPermissionedMint.sol](https://github.com/code-423n4/2022-09-frax/blob/main/src/ERC20/ERC20PermitPermissionedMint.sol)
* **openzeppelin-contracts/contracts/token/ERC20/IERC20.sol**
  * [src/ERC20/ERC20PermitPermissionedMint.sol](https://github.com/code-423n4/2022-09-frax/blob/main/src/ERC20/ERC20PermitPermissionedMint.sol)
  * [src/frxETHMinter.sol](https://github.com/code-423n4/2022-09-frax/blob/main/src/frxETHMinter.sol)
* **solmate/mixins/ERC4626.sol**
  * [lib/ERC4626/src/xERC4626.sol](https://github.com/code-423n4/2022-09-frax/blob/main/lib/ERC4626/src/xERC4626.sol)
* **solmate/utils/SafeCastLib.sol**
  * [lib/ERC4626/src/xERC4626.sol](https://github.com/code-423n4/2022-09-frax/blob/main/lib/ERC4626/src/xERC4626.sol)


# Contracts Under Review
## src/frxETH.sol
Simply wraps ERC20PermitPermissionedMint.sol, no logic to audit
## src/ERC20/ERC20PermitPermissionedMint.sol
Parent contract for frxETH.sol. Has EIP-712/EIP-2612 permit capability, is burnable, and has an array of authorized minters. Fork of the FPI contract which itself is a fork of the Frax contract so it is unlikely bugs are found here. Owned by the Frax multisig.
## src/sfrxETH.sol
Autocompounding vault token for frxETH. Users deposit their frxETH for sfrxETH. Adheres to ERC4626/xERC4626. Any rewards earned externally, such as from ETH 2.0 staking, are converted to frxETH then sent here. After that happens, the exchange rate / pricePerShare for the sfrxETH token increases and sfrxETH hodlers can exchange their sfrxETH tokens for more frxETH than they initially put in (this is their rewards). Has EIP-712/EIP-2612 permit capability. Inherits from xERC4626 for the majority of its important logic.
## lib/ERC4626/src/xERC4626.sol
Minimally forked from a Fei contract, which was inspired by Zefram Lou's xERC20, which was inspired by xSUSHI. A staking pool where the rewards token is the same as the stake token. Rewards sent to the vault are distributed over time to prevent malicious actors from stealing the rewards. Relies on an increasing exchange rate between sfrxETH and frxETH.
## src/frxETHMinter.sol
Authorized minter for frxETH. Users deposit ETH for frxETH. It then deposits that ETH into ETH 2.0 staking validators to earn yield. It can also withhold part of the ETH deposit for future use, such as to earn yield in other places to supplement the ETH 2.0 staking yield. Inherits from the OperatorRegistry to hold the parameters needed when depositing to the ETH 2.0 deposit contract.
## src/OperatorRegistry.sol
Keeps track of available validators to add batches of 32 ETH to. Each validator is represented by a struct containing the information needed when sending information to the ETH 2.0 deposit contract, save for withdrawal credential, which is global. Contains various array manipulations. It is assumed that validator checks for validity, as well as ordering in the array, are done off-chain to save gas beforehand, and are also built into the official deposit contract.


# Contracts Included but not under review
## ERC4626.sol
Solmate implementation, has been previously audited for 
## DepositContract.sol
Official Ethereum 2.0 deposit contract.
## Owned.sol
Synthetix.io created.
## SigUtils.sol
Only used for testing, so no need to audit.
## ERC20Permit, ERC20Burnable, etc
Openzeppelin created contracts are already extensively audited.

# Known Issues
### xERC4626.sol
xTRIBE, which is xERC4626.sol based and has some functions that sfrxETH.sol uses, has two known medium severity corner case issues [M-01](https://code4rena.com/reports/2022-04-xtribe/#m-01-xerc4626sol-some-users-may-not-be-able-to-withdraw-until-rewardscycleend-the-due-to-underflow-in-beforewithdraw) and 
[M-02](https://github.com/code-423n4/2022-04-xtribe-findings/issues/66)
### OperatorRegistry.sol
No checking for valid validator pubkeys, signatures, etc are done here. They are assumed to be done off chain before they are added. However, the official ETH 2.0 DepositContract.sol DOES do checks and will revert if something is wrong. It is assumed that the team, or the manager(s) of the OperatorRegistry.sol contract will remove/replace the offending invalid validators.

# Areas for Wardens to focus on:
Since frxETH.sol, and its inherited ERC20PermitPermissionedMint.sol are simple forks of previously audited work, it is unlikely bugs can be found in it. The most important areas of focus are therefore xERC4626.sol, which is the main logic of sfrxETH.sol, and frxETHMinter.sol. Those are where funds are handled, and therefore the most critical for security. OperatorRegistry.sol, which is inherited by the frxETHMinter, is mostly permissioned, so it is less likely to have vulnerabilities, but it is still useful to ensure malicious users cannot grief the stack of validator information. 

# Additional Information
```
- Protocol Name: Frax Finance, Frax Staked Ether
- Link to public Repository: https://github.com/FraxFinance/frxETH-public
- Number of (non-library) contracts in the scope: 5
- Total sLoC: 365
- How many library dependencies?   0
- How many separate interfaces and struct definitions for the contracts within scope?   1
- Does most of your code generally use composition or inheritance?   Yes
- How many external calls?   1
- Relevant context to audit this part of the protocol: Need to understand ETH 2.0 staking basics (validators, etc), how the ETH 2.0 deposit contract functions, as well as ERC4626/xERC4626 and EIP-712/EIP-2612
- Does it use an oracle?   No
- ERC20 compliant?   Yes for frxETH and sfrxETH
- Novel or unique curve logic or mathematical models?   No
- Does it use a timelock function?   Yes
- Is it an NFT?   No
- Does it have an AMM?   No
- Is it a fork of a popular project?   No
- Does it use rollups?   No
- Is it multi-chain?   No
- Does it use a side-chain?   No
- Preferred timezone for communication?   PST, Los Angeles
```

# Contact Us:
Jack Corddry, Discord: Jack Corddry#7070, Telegram @corddry, Twitter @Jackcorddry
Travis Moore, Discord: Travis | Frax Finance (¤³, ¤³)#1337, Telegram @FortisFortuna_89
