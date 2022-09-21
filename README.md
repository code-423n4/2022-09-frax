# Frax Ether Liquid Staking contest details
- $28,500 USDC main award pot
- $1,500 USDC gas optimization award pot
- Join [C4 Discord](https://discord.gg/code4rena) to register
- Submit findings [using the C4 form](https://code4rena.com/contests/2022-09-frax-contest/submit)
- [Read our guidelines for more details](https://docs.code4rena.com/roles/wardens)
- Starts September 22, 2022 20:00 UTC
- Ends September 25, 2022 20:00 UTC

# Frax Staked Ethereum
## Flowchart
![frxETH Flowchart](flowchart.svg)
## Overview Documentation
[https://docs.frax.finance/frax-ether/overview](https://docs.frax.finance/frax-ether/overview).


<!-- //////////////////////////////////////////////////////////////// -->
# Building and Testing
## Setup
1) Install [foundry](https://book.getfoundry.sh/getting-started/installation)
2) ```forge install```
3) ```git submodule update --init --recursive```
4) (Optional) Occasionally update / pull your submodules to keep them up to date. ```git submodule update --recursive --remote```
5) Create your own .env and copy SAMPLE.env into there. Sample mainnet validator deposit keys are in test/deposit_data-TESTS-MAINNET.json if you need more.
6) You don't need to add PRIVATE_KEY, ETHERSCAN_KEY, or FRXETH_OWNER if you are not actually deploying on live mainnet

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

If you need to fork mainnet, single test contract
```source .env && forge test --fork-url $MAINNET_RPC_URL -vv --match-path ./test/frxETHMinter.t.sol```

Verbosely test a single contract while forking mainnet
or ```source .env && forge test --fork-url $MAINNET_RPC_URL -m test_frxETHMinter_submitAndDepositRegular -vvvvv``` for single test verbosity level 5

### Slither
1) Install [slither](https://github.com/crytic/slither#how-to-install)
2) Slither a single contract
```slither ./src/sfrxETH.sol --solc-remaps "openzeppelin-contracts=lib/openzeppelin-contracts ERC4626=lib/ERC4626/src solmate=lib/solmate/src"```


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

## Development
To make new dependencies play nicely with VSCode:
```forge remappings > remappings.txt```

<!-- //////////////////////////////////////////////////////////////// -->
# Contracts Under Review
## src/frxETH.sol (43 loc)
Simply wraps ERC20PermitPermissionedMint.sol, no logic to audit
## src/ERC20/ERC20PermitPermissionedMint.sol (107 loc)
Parent contract for frxETH.sol. Has EIP-712/EIP-2612 permit capability, is burnable, and has an array of authorized minters. Fork of the FPI contract which itself is a fork of the Frax contract so it is unlikely bugs are found here. Owned by the Frax multisig.
## lib/ERC4626/src/xERC4626.sol (102 loc)
Minimally forked from a Fei contract, which was inspired by Zefram Lou's xERC20, which was inspired by xSUSHI. A staking pool where the rewards token is the same as the stake token. Rewards sent to the vault are distributed over time to prevent malicious actors from stealing the rewards. Relies on an increasing exchange rate between sfrxETH and frxETH.
## src/frxETHMinter.sol (213 loc)
Authorized minter for frxETH. Users deposit ETH for frxETH. It then deposits that ETH into ETH 2.0 staking validators to earn yield. It can also withhold part of the ETH deposit for future use, such as to earn yield in other places to supplement the ETH 2.0 staking yield.
## src/OperatorRegistry.sol (216 loc)
Keeps track of available validators to add batches of 32 ETH to. Contains various array manipulations. It is assumed that validator checks for validity, as well as ordering in the array, are done off-chain to save gas beforehand, and are also built into the official deposit contract.
## src/sfrxETH.sol (90 loc)
Autocompounding vault token for frxETH. Users deposit their frxETH for sfrxETH. Adheres to ERC4626/xERC4626. Any rewards earned externally, such as from ETH 2.0 staking, are converted to frxETH then sent here. After that happens, the exchange rate / pricePerShare for the sfrxETH token increases and sfrxETH hodlers can exchange their sfrxETH tokens for more frxETH than they initially put in (this is their rewards). Has EIP-712/EIP-2612 permit capability.

<!-- //////////////////////////////////////////////////////////////// -->
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

<!-- //////////////////////////////////////////////////////////////// -->
# Known Issues
### xERC4626.sol
xTRIBE, which is xERC4626.sol based and has some functions that sfrxETH.sol uses, has two known medium severity corner case issues [M-01](https://code4rena.com/reports/2022-04-xtribe/#m-01-xerc4626sol-some-users-may-not-be-able-to-withdraw-until-rewardscycleend-the-due-to-underflow-in-beforewithdraw) and 
[M-02](https://github.com/code-423n4/2022-04-xtribe-findings/issues/66)
### OperatorRegistry.sol
No checking for valid validator pubkeys, signatures, etc are done here. They are assumed to be done off chain before they are added. However, the official ETH 2.0 DepositContract.sol DOES do checks and will revert if something is wrong. It is assumed that the team, or the manager(s) of the OperatorRegistry.sol contract will remove/replace the offending invalid validators.

# Contact Us:
Jack Corddry, Discord: Jack Corddry#7070, Telegram @corddry, Twitter @Jackcorddry
Travis Moore, Discord: Travis | Frax Finance (¤³, ¤³)#1337, Telegram @FortisFortuna_89