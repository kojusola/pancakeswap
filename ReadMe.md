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

<a id="CakeToken"></a>`import "./CakeToken.sol"`

- This is a BEP20 token contract with governance.
- `mint()` function is used in this contract and it mints cakeTokens to the caller address and move delegates to the caller address from address(0) with the \_amount minted.

---

<a id="SyrupBar"></a>`import "import "./SyrupBar.sol"`

- This is a BEP20 token contract with governance.
- `mint()` function is used in this contract and it mints cakeTokens to the caller address and move delegates to the caller address from address(0) with the \_amount minted.
- `burn()` function is used in this contract and it burns cakeTokens to the caller address and move delegates from the caller address from address(0) to the \_amount minted.

---

<a id="IMigratorChef"></a>`interface IMigratorChef`

- This is an interface of a contract used to perform LP token migration from legacy PancakeSwap to CakeSwap.
- It contains only a function `migrate()` which returns the new address of the LP token. it has some rules:
  - Migrator must have allowance access to PancakeSwap LP tokens.
  - CakeSwap must mint exactly the same amount of CakeSwap LP tokens.

---

# Contract `MasterChef`

- `contract MasterChef is Ownable` => It inherits the [`Ownable`](#Ownable) contract. The ownership will be transferred to a governance smart contract once CAKE is sufficiently distributed and the community can show to govern itself
- `using SafeMath for uint256` => It attaches functions from library [`SafeMath`](#SafeMath) to `uint256`
- ` using SafeBEP20 for IBEP20` => It attaches functions from library [`SafeBEP20`](#SafeBEP20) to [`IBEP20`](#IBEP20)

## Data Structure

<a id="UserInfo"></a>

##

- `UserInfo`: This is a struct that contains details of each User. It includes:
  - `amount` => amount of LP tokens the user has provided
  - `rewardDebt` => amount of reward tokens user has withdrawn
  ##
       struct UserInfo {
        uint256 amount;
        uint256 rewardDebt;
        }

---

<a id="PoolInfo"></a>

##

- `PoolInfo`: This is a struct that contains details of each User. It includes:
  - `lpToken` => Address of LP token contract.
  - `allocPoint` => How many allocation points are assigned to this pool. CAKEs to distribute per block.
  - `lastRewardBlock` => Last block number that CAKEs distribution occurs.
  - `accCakePerShare` => Accumulated CAKEs per share, times 1e12. See below.
  ##
        struct PoolInfo {
        IBEP20 lpToken;
        uint256 allocPoint;
        uint256 lastRewardBlock;
        uint256 accCakePerShare;
        }

---

<a id="cake"></a>`CakeToken public cake`

- Initializes the CakeToken contract as cake.

---

<a id="syrup"></a>`SyrupBar public syrup`

- Initializes the SyrupBar contract as syrup.

---

<a id="devaddr"></a>`address public devaddr`

- Address of Developer.

---

<a id="cakePerBlock"></a>`uint256 public cakePerBlock`

- CAKE tokens created per block.

---

<a id="bonus"></a>`uint256 public BONUS_MULTIPLIER = 1`

- Bonus mulTiplier for early cake makers.

---

<a id="migrator"></a>`IMigratorChef public migrator`

- Initializing Migrator contract as migrator.

---

<a id="APoolInfo"></a>`PoolInfo[] public poolInfo`

- Array of all [`poolInfo`](#poolInfo)

---

<a id="mapp"></a>`mapping (uint256 => mapping (address => UserInfo)) public userInfo`

- A mapping of \_pid(pool id) to a mapping of address of msg.sender to [`UserInfo`](#UserInfo)

---

<a id="totalAllocPoint"></a>`uint256 public totalAllocPoint = 0`

- Total allocationpoints for all the pools

---

<a id="startBlock"></a>`uint256 public startBlock`

- The block number when CAKE mining starts.

---

## Events

<a id="Deposit"></a>`event Deposit(address indexed user, uint256 indexed pid, uint256 amount)`

- This returns the user address that deposited, pid(pool id) of depositor, amount deposited.

---

<a id="Withdraw"></a>`event Withdraw(address indexed user, uint256 indexed pid, uint256 amount)`

- This returns the user address that withdraws, pid(pool id) of widthdrawer, amount widthdrawn.

---

<a id="EmergencyWithdraw"></a>` event EmergencyWithdraw(address indexed user, uint256 indexed pid, uint256 amount)`

- This returns the user address that withdraws, pid(pool id) of widthdrawer, amount widthdrawn.

---
