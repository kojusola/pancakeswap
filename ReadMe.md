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

- Contract link => https://github.com/pancakeswap/pancake-farm/blob/master/contracts/MasterChef.sol

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
- `safeCakeTransfer` function is used just in case if rounding error causes pool to not have enough cake.

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

## Constructor

     constructor(
        CakeToken _cake,
        SyrupBar _syrup,
        address _devaddr,
        uint256 _cakePerBlock,
        uint256 _startBlock
        )

- This constructor takes in the following:
  - `_cake` => CakeToken,
  - `_syrup` => SyrupBar,
  - `_devaddr` => developer Address,
  - `_cakePerBlock` => Cake Per Block,
  - `_startBlock` => block number when CAKE mining starts,
  ##
- It sets all the above into their corresponding defined variables.
- It creates the first [`poolInfo`](#PoolInfo) struct which is the CakeToken with pid of 0 and pushed into the [`poolInfo Array`](#APoolInfo).
- It also allocates the [`totalAllocPoint`](#totalAllocPoint) 1000.

## Functions

updateMultiplier(uint256 multiplierNumber)

- This public function takes in the following:
  - `multiplierNumber` => multiplier number
  ##
- It updates the [`BONUS_MULTIPLIER`](#bonus)

##

poolLength()

- this external view function returns the length of the [`PoolInfo Array`](#APoolInfo).

##

add(uint256 \_allocPoint, IBEP20 \_lpToken, bool \_withUpdate)

- This public function adds a new lp token to the pool in the poolInfo array and takes in the following :
  - `_allocPoint` => allocPoint of the pool to be added,
  - `_lpToken` => lp token,
  - `_withUpdate` => whether to update all pools or not ( it's a boolean),
  ##
- the [`onlyOwner`](#Ownable) modifier ensures it can only be called by the owner of the contract ie the address that deploys it.
- if `_withUpdate` is true, The [`massUpdatePools()`](#massUpdatePools) function would be called.
- a variable `lastRewardBlock` which is a uint256 is defined an a ternary operator is used to check if block.number is greater than start block and if true the block.number is assigned but if false the startBlock is assigned.
- the [`totalAllocPoint`](#totalAllocPoint) is updated by adding the pool's `allocPoint`.
- it creates the [`poolInfo`](#PoolInfo) struct and pushed it into the [`poolInfo Array`](#APoolInfo).
- it also calls the [`updateStakingPool()`](#updateStakingPool) function.

##

set(uint256 \_pid, uint256 \_allocPoint, bool \_withUpdate)

- This public function updates the given lp token's cake allocation point and takes in the following :
  - `_pid` => pool id meaning position in the [`poolInfo Array`](#APoolInfo),
  - `_allocPoint` => new pool's allocation point,
  - `_withUpdate` => whether to update all pools or not ( it's a boolean),
  ##
- the [`onlyOwner`](#Ownable) modifier ensures it can only be called by the owner of the contract ie the address that deploys it.
- if `_withUpdate` is true, The [`massUpdatePools()`](#massUpdatePools) function would be called.
- a variable `prevAllocPoint` which is a uint256 is defined and equal to allocation point of the given pid using `poolInfo[_pid].allocPoint`
- the allocation point of the given pid (poolInfo[_pid].allocPoint) is then updated with new allocation point(`_allocPoint`)
- then conditional which is run when the previous allocation points (`prevAllocPoint`) is not equal to new allocation point(`_allocPoint`). The following is done if true:
  - the [`totalAllocPoint`](#totalAllocPoint) is updated by adding the pool's `allocPoint`and subtracting the previous allocation points (`prevAllocPoint`)
  - it also calls the [`updateStakingPool()`](#updateStakingPool) function.

<a id="updateStakingPool"></a>

##

updateStakingPool()

- This internal function updates the allocation point of cake token pool
  ##
- it gets the length of the [`PoolInfo Array (poolInfo)`](#APoolInfo) and defines it has `length`
- defines the variable `points` and initializes it has 0
- then, it loops through [`PoolInfo Array (poolInfo)`](#APoolInfo) adding each token's allocation points into variable points `points = points.add(poolInfo[pid].allocPoint)`, it starts from one to ensure only LP token allocation points are added together.
- finally, it ensures `points` is greater than 1 before dividing points by 3 `points = points.div(3)`, updating [`totalAllocPoint`](#totalAllocPoint) by subtracting allocation points for the cake token and adding the new points value, then updates the allocation points for cake token (`poolInfo[0].allocPoint = points`).

##

setMigrator(IMigratorChef \_migrator)

- This public function sets the migrator contract and takes in the following :
  - `_migrator` => migrator
  ##
- the [`onlyOwner`](#Ownable) modifier ensures function can only be called by the owner of the contract ie the address that deploys it.
- sets the [`migrator`](#migrator) to `_migrator`

  ##

migrate(uint256 \_pid)

- This public function migrates lp token to another lp contract and takes in the following :
  - `_pid` => pool id meaning position in the [`poolInfo Array`](#APoolInfo),
  ##
- requires that address of migrator has been set earlier and is not address(0)
- if `_withUpdate` is true, The `massUpdatePools()` function would be called.
- it gets the poolInfo of the token using `_pid`(`poolInfo[_pid]`) given from [`PoolInfo Array (poolInfo)`](#APoolInfo) and stores it in `pool`.
- it checks the lp token balance of the contract (`lpToken.balanceOf(address(this))`) and equates it to variable `bal`.
- then, the contract approves the migrator address to spend the lpToken on it's behalf then then the migrator contract returns the new address of the new lp token and instantiate the contract (`IBEP20 newLpToken = migrator.migrate(lpToken)`).
- The new token balance of the contract is required to be equal to `bal`, if not it reverts.
- then the poolInfo is updated with the newLpToken `pool.lpToken = newLpToken`.

<a id="getMultiplier"></a>

##

getMultiplier(uint256 \_from, uint256 \_to)

- This public view function calculates the BONUS_MULTIPLIER and takes in the following :
  - `_from` => the from block number
  - `_to` => the to block number
  ##
- it returns the result of `_to` minus `_from` multiplied by `BONUS_MULTIPLIER`

  ##

pendingCake(uint256 \_pid, address \_user)

- This view external function to see the pending cakes for an address's LP token and takes in the following :
  - `_pid` => LP token pool id.
  - `_user` => users address
  ##
- it gets the poolInfo for the given LP token from the [`PoolInfo Array (poolInfo)`](#APoolInfo) as `pool` and userInfo from the [`userInfo`](#mapp) mapping using LP token pool id and user's address as `user`.
- it gets the pool's accCakePerShare (`uint256 accCakePerShare = pool.accCakePerShare`)
- it also gets the amount LP token the contract has `uint256 lpSupply = pool.lpToken.balanceOf(address(this))`.
- then, a conditional that checks whether block.number is greater than pool.lastRewardBlock and the contracts LP token balance is greater than 0. if true, the following happens:
  - get the `multiplier` using the [`getMultiplier()`](#getMultiplier) function
  - calculate `cakeReward` by multiplying multiplier by [`cakePerBlock`](#cakePerBlock) and multiply by lp token allocation point (`pool.allocPoint`) then divide the total by [`totalAllocPoint`](#totalAllocPoint)
  - accCakePerShare of the lp token is updated by adding (`cakeReward` multiplied by 1e12) the dividing by the amount LP token the contract
- then returns the amount the user deposited multiplied by accCakePerShare(updated or not depending) divided by 1e12 and subtract the user's rewardDebt (which is the amount the user has withdrawn earlier).

<a id="massUpdatePools"></a>

##

massUpdatePools()

- This public function updates reward variables for all pools.
  ##
- it gets the length of the [`PoolInfo Array (poolInfo)`](#APoolInfo) and loops through it, for each of the PoolInfo it calls the [`updatePool()`](#updatePool) function.

<a id="updatePool"></a>

##

updatePool(uint256 \_pid)

- This public function updates reward variables for given pool and takes in the following:
  - `_pid` => LP token pool id.
  ##
- it gets the poolInfo for the given LP token from the [`PoolInfo Array (poolInfo)`](#APoolInfo) as `pool`.
- if block.number is less than or equal to the given lp Token lastRewardBlock, function returns.
- if the contract does not have any of the given lp token, the lastRewardBlock for the LP token is updated to block.number and returned.
- get the `multiplier` using the [`getMultiplier()`](#getMultiplier) function
- calculate `cakeReward` by multiplying multiplier by [`cakePerBlock`](#cakePerBlock) and multiply by lp token allocation point (`pool.allocPoint`) then divide the total by [`totalAllocPoint`](#totalAllocPoint)
- then cake token contracts mints `cakeReward` to [`syrup`](#syrup) contract and 10% of `cakeReward` to [`devaddr`](#devaddr) (to be burnt by the [`devaddr`](#devaddr) )
- accCakePerShare of the lp token is updated by adding (`cakeReward` multiplied by 1e12) the dividing by the amount LP token the contract.
- finally, update the LP token lastRewardBlock to block.number.

##

deposit(uint256 \_pid, uint256 \_amount)

- This public function deposits LP tokens for CAKE allocation and takes in the following:
  - `_pid` => LP token pool id.
  - `_amount` => amount of lp tokens to deposit.
  ##
- requires that `_pid` is not equal to 0, since `_pid` equal to 0 is cake staking not LP tokens deposits.
- it gets the poolInfo for the given LP token from the [`PoolInfo Array (poolInfo)`](#APoolInfo) as `pool` and userInfo from the [`userInfo`](#mapp) mapping using LP token pool id and msg.sender as `user`.
- it calls the [`updatePool`](#updatePool) function to ensure the rewards are upto date.
- if the user has staked before, then we calculate the amount of rewards pending(user's amount multiplied by lp tokens accCakePerShare, divided by 1e12 and the users rewardDebt is subtracted), then if rewards are pending they are transferred to the msg.sender using[`safeCakeTransfer(msg.sender, pending)`](#safeCakeTransfer) function.
- if `_amount` is greater than zero, then the `_amount` lpToken is transferred from the msg.sender to contract and update tha users amount by adding `_amount`.
- user's rewardDebt is replaced by user's updated amount multiplied by lp tokens accCakePerShare, divided by 1e12.
- finally emits the [`Deposit`](#Deposit) event.

##

withdraw(uint256 \_pid, uint256 \_amount)

- This public function withdraws LP tokens and takes in the following:
  - `_pid` => LP token pool id.
  - `_amount` => amount of lp tokens to withdraw.
  ##
- requires that `_pid` is not equal to 0, since `_pid` equal to 0 is cake pool.
- it gets the poolInfo for the given LP token from the [`PoolInfo Array (poolInfo)`](#APoolInfo) as `pool` and userInfo from the [`userInfo`](#mapp) mapping using LP token pool id and msg.sender as `user`.
- it requires the `_amount`is less than the amount of LP tokens the user has in the contract
- it calls the [`updatePool`](#updatePool) function to ensure the rewards are upto date.
- then we calculate the amount of rewards pending(user's amount multiplied by lp tokens accCakePerShare, divided by 1e12 and the users rewardDebt is subtracted), then if rewards are pending (ie amount of rewards pending is greater than 0) they are transferred to the msg.sender using [`safeCakeTransfer(msg.sender, pending)`](#safeCakeTransfer) function.
- if `_amount` is greater than zero, update tha users amount by subtracting `_amount` and then the `_amount` lpToken is transferred from the contract to msg.sender.
- user's rewardDebt is replaced by user's updated amount multiplied by lp tokens accCakePerShare, divided by 1e12.
- finally emits the [`Withdraw`](#Withdraw) event.

##

enterStaking(uint256 \_amount)

- This public function stakes cake tokens and takes in the following:
  - `_amount` => amount of cake tokens to deposit.
  ##
- the `_pid` is equal to 0 is cake token staking.
- it gets the poolInfo for the cake tokens from the [`PoolInfo Array (poolInfo)`](#APoolInfo) as `pool` and userInfo from the [`userInfo`](#mapp) mapping using cake token pool id and msg.sender as `user`.
- it requires the `_amount` is less than the amount of cake tokens the user has in the contract.
- it calls the [`updatePool`](#updatePool) function to ensure the rewards are upto date.
- if the user has staked before, then we calculate the amount of rewards pending(user's amount multiplied by cake token accCakePerShare, divided by 1e12 and the users rewardDebt is subtracted), then if rewards are pending they are transferred to the msg.sender using[`safeCakeTransfer(msg.sender, pending)`](#safeCakeTransfer) function.
- if `_amount` is greater than zero, then the `_amount` cakeToken is transferred from the msg.sender to contract and update tha users amount by adding `_amount`.
- user's rewardDebt is replaced by user's updated amount multiplied by cake tokens accCakePerShare, divided by 1e12.
- then the syrup contract calls the `mint()` function from the [`SyrupBar`](#SyrupBar)
- finally emits the [`Deposit`](#Deposit) event.

##

leaveStaking(uint256 \_amount)

- This public function unstakes cake tokens and takes in the following:
  - `_amount` => amount of cake tokens to withdraw.
  ##
- the `_pid` is equal to 0 is cake token staking.
- it gets the poolInfo for the cake tokens from the [`PoolInfo Array (poolInfo)`](#APoolInfo) as `pool` and userInfo from the [`userInfo`](#mapp) mapping using cake token pool id and msg.sender as `user`.
- it calls the [`updatePool`](#updatePool) function to ensure the rewards are upto date.
- if the user has staked before, then we calculate the amount of rewards pending(user's amount multiplied by cake token accCakePerShare, divided by 1e12 and the users rewardDebt is subtracted), then if rewards are pending they are transferred to the using [`safeCakeTransfer(msg.sender, pending)`](#safeCakeTransfer) function.
- if `_amount` is greater than zero, update tha users amount by subtracting `_amount` and then the `_amount` cakeToken is transferred from the contract to msg.sender.
- user's rewardDebt is replaced by user's updated amount multiplied by cake tokens accCakePerShare, divided by 1e12.
- then the syrup contract calls the `burn()` function from the [`SyrupBar`](#SyrupBar)
- finally emits the [`Deposit`](#Deposit) event.

##

emergencyWithdraw(uint256 \_pid)

- This public function unstakes cake tokens without care for rewards and takes in the following:
  - `_pid` => token pool id.
  ##
- it gets the poolInfo for the given token from the [`PoolInfo Array (poolInfo)`](#APoolInfo) as `pool` and userInfo from the [`userInfo`](#mapp) mapping using token pool id and msg.sender as `user`.
- all the given token the msg.sender has is transferred from the contract to msg.sender.
- emits the [`EmergencyWithdraw`](#EmergencyWithdraw) event.
- then the user's amount and user's rewardDebt is equated to zero
- Although, this function also allows for cake token to be withdrawn ( ie unstaking) without burning the syrup token already minted. this can be exploited by some users.

<a id="safeCakeTransfer"></a>

##

safeCakeTransfer(address \_to, uint256 \_amount)

- This public function transfers cake tokens just in case rounding error causes pool to not have enough cake and takes in the following:
  - `_to` => address to.
  - `_amount` => amount of cake token to send.
  ##
- it transfers cake tokens.

##

dev(address \_devaddr)

- This public function updates dev address by the previous dev and takes in the following:
  - `_devaddr` => new dev address.
  ##
- it requires that the msg.sender is the present [`devaddr`](#devaddr)
- then replaces the old devaddr with `_devaddr`.
