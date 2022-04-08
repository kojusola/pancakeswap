# Pancakeswap Pancake Farm MasterChef Contract Review

---

# Introduction

Pancakeswap is a Binance Smart Chain-based (BSC) Decentralized Exchange (DEX) launched by anonymous developers. It is a DEX for swapping BEP-20 tokens. It uses automated market maker (AMM) model meaning that while you can trade digital assets on the platform, you trade against a liquidity pool.
Liquidity pools are filled with other user’s funds, when they deposit into the pool, they get liquidity provider tokens (lp tokens) in return. They can use these tokens to reclaim their share plus a portion of the trading fees i.e. users can trade BEP-20 tokens or add liquidity and earn rewards. For example, if a user adds BETH and ETH, the user gets BETH-ETH lp tokens.
Pancakeswap also allows users to farm its token (CAKE). Users can deposit their lp tokens, locking them up in a process that rewards them with cake which is the farming process. Users can then earn more by staking their CAKE

---

# Structure of Pancake Farm Repository

The pancake-farm repo contains the smart contracts for farms and pools, cake token and lottery reward. The Cake Token is the native token used in PancakeSwap. The SyrupBar token is the governance token.
MasterChef contract contains code to handle staking to earn Cake. When MasterChef contract is instantiated, it creates a default Cake pool. MasterChef contract contains code to add additional LP token to pool (function add), deposit LP tokens to pool (function deposit), withdraw LP tokens from pool (function withdraw), add(stake) Cake to pool (function enterStaking), remove(stake) Cake from pool (function leaveStaking), and remove(stake) Cake from pool without considering rewards (function emergencyWidthdraw). When you stake Cake, you get Syrup tokens.
SousChef contract handles staking Syrup token (aka SyrupBar token) to earn reward. The Syrup token is a kind of LP token, as well as a BEP20 token. The reward can be in Cake token or other tokens.

---

# MasterChef Contract

# Libraries and External Contracts Used

<a id="SafeMath"></a>`import "@pancakeswap/pancake-swap-lib/contracts/math/SafeMath.sol"`

- This is a library used to perform basic mathematical operations on Ethereum contracts and used for uint256 in the contract. Since the MasterChef contract was built with solidity version 0.6.12. SafeMath is added to validate mathematical operations (between two uint256 parameters in this case) to check if it would result in an integer overflow/underflow and throws an exception, effectively reverting the transaction. It contains different functions but the most used in this contract are add (to add them), sub(to subtract them), mul (to multiply them) and div (to divide them).

---

<a id="IBEP20"></a>`import '@pancakeswap/pancake-swap-lib/contracts/token/BEP20/IBEP20.sol'`

- Interface of the BEP20 (Binance Smart Chain Evolution Proposal 20) which is a token standard on BSC similar to the ERC20 token on ethereum. It has quite a few standard functions and events defined, It is used in this contract to initialize the lp Token contracts.

---

<a id="SafeBEP20"></a>`import '@pancakeswap/pancake-swap-lib/contracts/token/BEP20/SafeBEP20.sol'`

- It is a library that wraps around BEP20 operations that is throws on failure ie when the token contract itself returns false. It helps to securely interact with existing tokens by calling function `_callOptionalReturn()` . It contains functions like safeTransfer(), safeTransferFrom(), safeApprove(), safeDecreaseAloowance(), safeIncreaseAllowance().

---

<a id="Ownable"></a>`import '@pancakeswap/pancake-swap-lib/contracts/access/Ownable.sol'`

- Contract module which provides a basic access control mechanism. The contract restricts assess by recording the msg.sender on deployment.

---

<a id="CakeToken">`import "./CakeToken.sol"`

- This is a BEP20 token contract with governance.
- `mint()` function is used in this contract and it mints cakeTokens to the caller address and move delegates to the caller address from address(0) with the \_amount minted.

---

<a id="SyrupBar">`import "import "./SyrupBar.sol"`

- This is a BEP20 token contract with governance.
- `mint()` function is used in this contract and it mints cakeTokens to the caller address and move delegates to the caller address from address(0) with the \_amount minted.
- `burn()` function is used in this contract and it burns cakeTokens to the caller address and move delegates from the caller address from address(0) to the \_amount minted.

---

<a id="IMigratorChef">`interface IMigratorChef`

- This is an interface of a contract used to perform LP token migration from legacy PancakeSwap to CakeSwap.
- It contains only a function `migrate()` which returns the new address of the LP token. it has some rules:
  - Migrator must have allowance access to PancakeSwap LP tokens.
  - CakeSwap must mint exactly the same amount of CakeSwap LP tokens.

---

# Contract `MasterChef`

It inherits the `Ownable` contract

# Data Structure
