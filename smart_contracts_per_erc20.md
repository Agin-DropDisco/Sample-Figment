# **Part II & III**
**This tutorial part should take you about 2 ~ 3 hour to complete**
## What you will be doing:
- [x]  Create, write & deploy smart contracts on polygon
- [x]  Create, test and Run unit-testing for smart contracts
- [x]  Interacts with smart contracts

# **How to create, write, deploy and interacts with smart contracts on polygon**
## **First: Create Periphery Smart Contracts**
Peripheral Smart Contracts used for interacting with DexSwap Dapp.

**Workflow**
<img src="https://gateway.pinata.cloud/ipfs/QmVtjSZNMsJSuk9ai7jyrixhmfoWiV8CEd1PVr6vvk9jMV" align="center">


* The Smart contract itself is fork of the [Uniswapv2 periphery smart contracts v1.0.0.](https://github.com/Uniswap/uniswap-v2-periphery/releases/tag/v1.0.0) with some necesary modification for DexSwap Dapp need.

* Clone smart contracts periphery template from github
```javascript
git clone https://github.com/Agin-DropDisco/figment-periphery.git
cd figment-periphery && yarn i
```
* After All the process is done lets move to another step

## **Second:Create New Javascript File**
### **2_deploy.js**
```javascript

const DexSwapFactory = artifacts.require("IDexSwapFactory");
const DexSwapRouter = artifacts.require("DexSwapRouter");
const WETH = artifacts.require("WETH");
const argValue = (arg, defaultValue) => (process.argv.includes(arg) ? process.argv[process.argv.indexOf(arg) + 1] : defaultValue);
const network = () => argValue("--network", "local");


// MATIC MAINNET
const FACTORY_MATIC = ""; // Address from Deployed Smart contract Core
const WMATIC_MATIC = "0x0d500B1d8E8eF31E21C99d1Db9A6444d3ADf1270"; // See https://github.com/maticnetwork/static/blob/master/network/mainnet/v1/index.json

// MATIC TESTNET
const FACTORY_MUMBAI = ""; // Address from Deployed Smart contract Core
const WMATIC_MUMBAI = ""; // See https://github.com/maticnetwork/static/blob/master/network/testnet/mumbai/index.json

module.exports = async (deployer) => {

    const BN = web3.utils.toBN;
    const bnWithDecimals = (number, decimals) => BN(number).mul(BN(10).pow(BN(decimals)));
    const senderAccount = (await web3.eth.getAccounts())[0];

    if (network() === "mumbai") {

        console.log();
        console.log(":: REUSE FACTORY");
        let DexSwapFactoryInstance = await DexSwapFactory.at(FACTORY_MUMBAI);
        console.log(`DEXSWAP FACTORY:`, DexSwapFactoryInstance.address);


        console.log();
        console.log(":: DEPLOY ROUTER");
        await deployer.deploy(DexSwapRouter, DexSwapFactoryInstance.address, WMATIC_MUMBAI);
        const DexSwapRouterInstance = await DexSwapRouter.deployed();
        console.log(`DEXSWAP ROUTER:`, DexSwapRouterInstance.address);


    } else if (network() === "matic") {

        console.log();
        console.log(":: REUSE FACTORY");
        let DexSwapFactoryInstance = await DexSwapFactory.at(FACTORY_MATIC);
        console.log(`DEXSWAP FACTORY:`, DexSwapFactoryInstance.address);

        console.log();
        console.log(":: DEPLOY ROUTER");
        await deployer.deploy(DexSwapRouter, DexSwapFactoryInstance.address, WMATIC_MATIC);
        const DexSwapRouterInstance = await DexSwapRouter.deployed();
        console.log(`DEXSWAP ROUTER:`, DexSwapRouterInstance.address);

    }
};
```

## **Third:Deploy the contracts to polygon using Truffle**
```bash
truffle migrate --network matic
```
### **Result**
```javascript
Starting migrations...
======================
> Network name:    'matic'
> Network id:      137
> Block gas limit: 20277704 (0x13569c8)


2_deploy_contracts.js
=====================

:: REUSE FACTORY
DEXSWAP FACTORY: 0xeE9E5959bC2332EF6f29AcF40758a27ADa9ce6Cb

:: DEPLOY ROUTER

   Deploying 'DexSwapRouter'
   -------------------------
   > transaction hash:    0x503cd883f7e8df3ebcd501955544b48927792f83c0a0d2644caa568545444982
   > Blocks: 2            Seconds: 5
   > contract address:    0x7e87a968141Ec02254B53B9e58f9D64355B11B18
   > block number:        16885871
   > block timestamp:     1626349584
   > account:             0xeED588Fc1A009aFf2f4E90b57D4d74B2F56309c5
   > balance:             5.992850874058530957
   > gas used:            3954267 (0x3c565b)
   > gas price:           2.1 gwei
   > value sent:          0 ETH
   > total cost:          0.0083039607 ETH

   Pausing for 2 confirmations...
   ------------------------------
   > confirmation number: 2 (block: 16885876)
DEXSWAP ROUTER: 0x7e87a968141Ec02254B53B9e58f9D64355B11B18

   > Saving migration to chain.
   > Saving artifacts
   -------------------------------------
   > Total cost:        0.0083039607 ETH
```
#### Source WMATIC Mainnet: ***https://github.com/maticnetwork/static/blob/master/network/mainnet/v1/index.json***

#### Source WMATIC Testnet: ***https://github.com/maticnetwork/static/blob/master/network/testnet/mumbai/index.json***




# **Create, Test and Run unit-testing for smart contracts**

## At this part we will *create ERC20 Staking Contracts* and *Staking Rewards Distributions Contracts* to facilitate our own Liquidity Minning Platform and to make more understand how to create, test and run unit testing of smart contracts.

- Workflow
<img src="https://gateway.pinata.cloud/ipfs/QmUGzgoxJyWaff4g2hjBnaiKUHvwqnNMWwY2prEm8tr6ez" align="center">

### **1. Create ERC20 Staking Rewards Distributions Contracts**
### **2. Create Staking Distributions Contracts**

## **1. Create ERC20 Staking Rewards Distributions Contracts**
A generic contracts suite to bootstrap staking campaigns in which stakers get distributed rewards over time in relation to their share of the total staked tokens. Supports multiple ERC20 reward tokens, locked campaigns (i.e. no withdrawals until the end of the distribution if tokens are staked), capped campaigns, and rewards recovery by the owner for those dead moments in which no tokens are staked.(see User Interaction Workflow)
### Installation {#installation}
```bash
mkdir ERC20-Staking-Rewards
cd ERC20-Staking-Rewards
truffle init
yarn init
```
**We need to install/add dependecies**
```
* openzeppelin/contracts
* commitlint/cli
* commitlint/config-conventional
* babel-eslint
* truffle
* husky
* eth-gas-reporter
* ganache
* prettier
* solidity-coverage
* solhint-plugin-prettier
* prettier-plugin-solidity
```
**eg:**
```bash
yarn add openzeppelin/contracts && yarn add truffle
```
### **Now go to the contracts folder and create new folder name *interfaces***
### or simply just type this command in your terminal

```bash
cd contracts
mkdir interfaces
cd interfaces
```

### Inside of **interfaces** folder we need to Create New Solidity File below:
### **IERC20StakingRewardsDistribution.sol**
```javascript
// SPDX-License-Identifier: GPL-3.0

pragma solidity ^0.8.0;

interface IERC20StakingRewardsDistribution {
    function rewardAmount(address _rewardToken) external view returns (uint256);

    function recoverableUnassignedReward(address _rewardToken)
        external
        view
        returns (uint256);

    function stakedTokensOf(address _staker) external view returns (uint256);

    function getRewardTokens() external view returns (address[] memory);

    function getClaimedRewards(address _claimer)
        external
        view
        returns (uint256[] memory);

    function initialize(
        address[] calldata _rewardTokenAddresses,
        address _stakableTokenAddress,
        uint256[] calldata _rewardAmounts,
        uint64 _startingTimestamp,
        uint64 _endingTimestamp,
        bool _locked,
        uint256 _stakingCap
    ) external;

    function cancel() external;

    function recoverUnassignedRewards() external;

    function stake(uint256 _amount) external;

    function withdraw(uint256 _amount) external;

    function claim(uint256[] memory _amounts, address _recipient) external;

    function claimAll(address _recipient) external;

    function exit(address _recipient) external;

    function consolidateReward() external;

    function claimableRewards(address _staker)
        external
        view
        returns (uint256[] memory);

    function renounceOwnership() external;

    function transferOwnership(address _newOwner) external;
}
```
### **IERC20StakingRewardsDistributionFactory.sol**
```javascript
// SPDX-License-Identifier: GPL-3.0

pragma solidity ^0.8.0;

interface IERC20StakingRewardsDistributionFactory {
    function createDistribution(
        address[] calldata _rewardTokenAddresses,
        address _stakableTokenAddress,
        uint256[] calldata _rewardAmounts,
        uint64 _startingTimestamp,
        uint64 _endingTimestamp,
        bool _locked,
        uint256 _stakingCap
    ) external;

    function getDistributionsAmount() external view returns (uint256);

    function implementation() external view returns (address);

    function upgradeTo(address newImplementation) external;

    function distributions(uint256 _index) external returns (address);

    function stakingPaused() external returns (bool);
}
```


### **Now go to back to the contracts folder and create new folder, name: test**
### **or simply just type this command in your terminal**

```bash
cd ..
mkdir test && cd test
```
### **Inside of the "test" folder create new file, TestDependencies.sol, and paste the code**
```javascript
//SPDX-License-Identifier: GPL-3.0

pragma solidity ^0.8.0;

import "@openzeppelin/contracts/token/ERC20/presets/ERC20PresetMinterPauser.sol";
import "../ERC20StakingRewardsDistribution.sol";

contract FirstRewardERC20 is ERC20PresetMinterPauser {
    constructor() ERC20PresetMinterPauser("First reward", "RWD1") {}
}

contract SecondRewardERC20 is ERC20PresetMinterPauser {
    constructor() ERC20PresetMinterPauser("Second reward", "RWD2") {}
}

contract ZeroDecimalsRewardERC20 is ERC20PresetMinterPauser {
    constructor() ERC20PresetMinterPauser("Zero decimals reward", "ZDRWD") {}

    function decimals() public pure override returns (uint8) {
        return 0;
    }
}

contract FirstStakableERC20 is ERC20PresetMinterPauser {
    constructor() ERC20PresetMinterPauser("First stakable", "STK1") {}
}

contract SecondStakableERC20 is ERC20PresetMinterPauser {
    constructor() ERC20PresetMinterPauser("Second stakable", "STK2") {}
}

contract UpgradedERC20StakingRewardsDistribution is
    ERC20StakingRewardsDistribution
{
    function isUpgraded() external pure returns (bool) {
        return true;
    }
}
```


### **And now let's go back to the contracts folder and create new file**

**1.ERC20StakingRewardsDistribution.sol**

**2.ERC20StakingRewardsDistributionFactory.sol**

### **contracts/ERC20StakingRewardsDistribution.sol**
```javascript
// SPDX-License-Identifier: GPL-3.0

pragma solidity ^0.8.0;

import "@openzeppelin/contracts/utils/math/Math.sol";
import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
import "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";
import "./interfaces/IERC20StakingRewardsDistributionFactory.sol";

/**
 * Errors codes:
 *
 * SRD01: invalid starting timestamp
 * SRD02: invalid time duration
 * SRD03: inconsistent reward token/amount
 * SRD04: 0 address as reward token
 * SRD05: no reward
 * SRD06: no funding
 * SRD07: 0 address as stakable token
 * SRD08: distribution already started
 * SRD09: tried to stake nothing
 * SRD10: staking cap hit
 * SRD11: tried to withdraw nothing
 * SRD12: funds locked until the distribution ends
 * SRD13: withdrawn amount greater than current stake
 * SRD14: inconsistent claimed amounts
 * SRD15: insufficient claimable amount
 * SRD16: 0 address owner
 * SRD17: caller not owner
 * SRD18: already initialized
 * SRD19: invalid state for cancel to be called
 * SRD20: not started
 * SRD21: already ended
 * SRD22: no rewards are recoverable
 * SRD23: no rewards are claimable while claiming all
 * SRD24: no rewards are claimable while manually claiming an arbitrary amount of rewards
 * SRD25: staking is currently paused
 */
contract ERC20StakingRewardsDistribution {
    using SafeERC20 for IERC20;

    uint224 constant MULTIPLIER = 2**112;

    struct Reward {
        address token;
        uint256 amount;
        uint256 perStakedToken;
        uint256 recoverableSeconds;
        uint256 claimed;
    }

    struct StakerRewardInfo {
        uint256 consolidatedPerStakedToken;
        uint256 earned;
        uint256 claimed;
    }

    struct Staker {
        uint256 stake;
        mapping(address => StakerRewardInfo) rewardInfo;
    }

    Reward[] public rewards;
    mapping(address => Staker) public stakers;
    uint64 public startingTimestamp;
    uint64 public endingTimestamp;
    uint64 public secondsDuration;
    uint64 public lastConsolidationTimestamp;
    IERC20 public stakableToken;
    address public owner;
    address public factory;
    bool public locked;
    bool public canceled;
    bool public initialized;
    uint256 public totalStakedTokensAmount;
    uint256 public stakingCap;

    event OwnershipTransferred(
        address indexed previousOwner,
        address indexed newOwner
    );
    event Initialized(
        address[] rewardsTokenAddresses,
        address stakableTokenAddress,
        uint256[] rewardsAmounts,
        uint64 startingTimestamp,
        uint64 endingTimestamp,
        bool locked,
        uint256 stakingCap
    );
    event Canceled();
    event Staked(address indexed staker, uint256 amount);
    event Withdrawn(address indexed withdrawer, uint256 amount);
    event Claimed(address indexed claimer, uint256[] amounts);
    event Recovered(uint256[] amounts);

    function initialize(
        address[] calldata _rewardTokenAddresses,
        address _stakableTokenAddress,
        uint256[] calldata _rewardAmounts,
        uint64 _startingTimestamp,
        uint64 _endingTimestamp,
        bool _locked,
        uint256 _stakingCap
    ) external onlyUninitialized {
        require(_startingTimestamp > block.timestamp, "SRD01");
        require(_endingTimestamp > _startingTimestamp, "SRD02");
        require(_rewardTokenAddresses.length == _rewardAmounts.length, "SRD03");

        secondsDuration = _endingTimestamp - _startingTimestamp;
        // Initializing reward tokens and amounts
        for (uint32 _i = 0; _i < _rewardTokenAddresses.length; _i++) {
            address _rewardTokenAddress = _rewardTokenAddresses[_i];
            uint256 _rewardAmount = _rewardAmounts[_i];
            require(_rewardTokenAddress != address(0), "SRD04");
            require(_rewardAmount > 0, "SRD05");
            IERC20 _rewardToken = IERC20(_rewardTokenAddress);
            require(
                _rewardToken.balanceOf(address(this)) >= _rewardAmount,
                "SRD06"
            );
            rewards.push(
                Reward({
                    token: _rewardTokenAddress,
                    amount: _rewardAmount,
                    perStakedToken: 0,
                    recoverableSeconds: 0,
                    claimed: 0
                })
            );
        }

        require(_stakableTokenAddress != address(0), "SRD07");
        stakableToken = IERC20(_stakableTokenAddress);

        owner = msg.sender;
        factory = msg.sender;
        startingTimestamp = _startingTimestamp;
        endingTimestamp = _endingTimestamp;
        lastConsolidationTimestamp = _startingTimestamp;
        locked = _locked;
        stakingCap = _stakingCap;
        initialized = true;
        canceled = false;

        emit Initialized(
            _rewardTokenAddresses,
            _stakableTokenAddress,
            _rewardAmounts,
            _startingTimestamp,
            _endingTimestamp,
            _locked,
            _stakingCap
        );
    }

    function cancel() external onlyOwner {
        require(initialized && !canceled, "SRD19");
        require(block.timestamp < startingTimestamp, "SRD08");
        for (uint256 _i; _i < rewards.length; _i++) {
            Reward storage _reward = rewards[_i];
            IERC20(_reward.token).safeTransfer(
                owner,
                IERC20(_reward.token).balanceOf(address(this))
            );
        }
        canceled = true;
        emit Canceled();
    }

    function recoverUnassignedRewards() external onlyStarted {
        consolidateReward();
        uint256[] memory _recoveredUnassignedRewards =
            new uint256[](rewards.length);
        bool _atLeastOneNonZeroRecovery = false;
        for (uint256 _i; _i < rewards.length; _i++) {
            Reward storage _reward = rewards[_i];
            // recoverable rewards are going to be recovered in this tx (if it does not revert),
            // so we add them to the claimed rewards right now
            _reward.claimed += ((_reward.recoverableSeconds * _reward.amount) /
                (uint256(secondsDuration) * MULTIPLIER));
            delete _reward.recoverableSeconds;
            uint256 _recoverableRewards =
                IERC20(_reward.token).balanceOf(address(this)) -
                    (_reward.amount - _reward.claimed);
            if (!_atLeastOneNonZeroRecovery && _recoverableRewards > 0)
                _atLeastOneNonZeroRecovery = true;
            _recoveredUnassignedRewards[_i] = _recoverableRewards;
            IERC20(_reward.token).safeTransfer(owner, _recoverableRewards);
        }
        require(_atLeastOneNonZeroRecovery, "SRD22");
        emit Recovered(_recoveredUnassignedRewards);
    }

    function stake(uint256 _amount) external onlyRunning {
        require(
            !IERC20StakingRewardsDistributionFactory(factory).stakingPaused(),
            "SRD25"
        );
        require(_amount > 0, "SRD09");
        if (stakingCap > 0) {
            require(totalStakedTokensAmount + _amount <= stakingCap, "SRD10");
        }
        consolidateReward();
        Staker storage _staker = stakers[msg.sender];
        _staker.stake += _amount;
        totalStakedTokensAmount += _amount;
        stakableToken.safeTransferFrom(msg.sender, address(this), _amount);
        emit Staked(msg.sender, _amount);
    }

    function withdraw(uint256 _amount) public onlyStarted {
        require(_amount > 0, "SRD11");
        if (locked) {
            require(block.timestamp > endingTimestamp, "SRD12");
        }
        consolidateReward();
        Staker storage _staker = stakers[msg.sender];
        require(_staker.stake >= _amount, "SRD13");
        _staker.stake -= _amount;
        totalStakedTokensAmount -= _amount;
        stakableToken.safeTransfer(msg.sender, _amount);
        emit Withdrawn(msg.sender, _amount);
    }

    function claim(uint256[] memory _amounts, address _recipient)
        external
        onlyStarted
    {
        require(_amounts.length == rewards.length, "SRD14");
        consolidateReward();
        Staker storage _staker = stakers[msg.sender];
        uint256[] memory _claimedRewards = new uint256[](rewards.length);
        bool _atLeastOneNonZeroClaim = false;
        for (uint256 _i; _i < rewards.length; _i++) {
            Reward storage _reward = rewards[_i];
            StakerRewardInfo storage _stakerRewardInfo =
                _staker.rewardInfo[_reward.token];
            uint256 _claimableReward =
                _stakerRewardInfo.earned - _stakerRewardInfo.claimed;
            uint256 _wantedAmount = _amounts[_i];
            require(_claimableReward >= _wantedAmount, "SRD15");
            if (!_atLeastOneNonZeroClaim && _wantedAmount > 0)
                _atLeastOneNonZeroClaim = true;
            _stakerRewardInfo.claimed += _wantedAmount;
            _reward.claimed += _wantedAmount;
            IERC20(_reward.token).safeTransfer(_recipient, _wantedAmount);
            _claimedRewards[_i] = _wantedAmount;
        }
        require(_atLeastOneNonZeroClaim, "SRD24");
        emit Claimed(msg.sender, _claimedRewards);
    }

    function claimAll(address _recipient) public onlyStarted {
        consolidateReward();
        Staker storage _staker = stakers[msg.sender];
        uint256[] memory _claimedRewards = new uint256[](rewards.length);
        bool _atLeastOneNonZeroClaim = false;
        for (uint256 _i; _i < rewards.length; _i++) {
            Reward storage _reward = rewards[_i];
            StakerRewardInfo storage _stakerRewardInfo =
                _staker.rewardInfo[_reward.token];
            uint256 _claimableReward =
                _stakerRewardInfo.earned - _stakerRewardInfo.claimed;
            if (!_atLeastOneNonZeroClaim && _claimableReward > 0)
                _atLeastOneNonZeroClaim = true;
            _stakerRewardInfo.claimed += _claimableReward;
            _reward.claimed += _claimableReward;
            IERC20(_reward.token).safeTransfer(_recipient, _claimableReward);
            _claimedRewards[_i] = _claimableReward;
        }
        require(_atLeastOneNonZeroClaim, "SRD23");
        emit Claimed(msg.sender, _claimedRewards);
    }

    function exit(address _recipient) external onlyStarted {
        claimAll(_recipient);
        withdraw(stakers[msg.sender].stake);
    }

    function consolidateReward() private {
        uint64 _consolidationTimestamp =
            uint64(Math.min(block.timestamp, endingTimestamp));
        uint256 _lastPeriodDuration =
            uint256(_consolidationTimestamp - lastConsolidationTimestamp);
        Staker storage _staker = stakers[msg.sender];
        for (uint256 _i; _i < rewards.length; _i++) {
            Reward storage _reward = rewards[_i];
            StakerRewardInfo storage _stakerRewardInfo =
                _staker.rewardInfo[_reward.token];
            if (_lastPeriodDuration > 0) {
                if (totalStakedTokensAmount == 0) {
                    _reward.recoverableSeconds +=
                        _lastPeriodDuration *
                        MULTIPLIER;
                    // no need to update the reward per staked token since in this period
                    // there have been no staked tokens, so no reward has been given out to stakers
                } else {
                    _reward.perStakedToken += ((_lastPeriodDuration *
                        _reward.amount *
                        MULTIPLIER) /
                        (totalStakedTokensAmount * secondsDuration));
                }
            }
            uint256 _rewardSinceLastConsolidation =
                (_staker.stake *
                    (_reward.perStakedToken -
                        _stakerRewardInfo.consolidatedPerStakedToken)) /
                    MULTIPLIER;
            if (_rewardSinceLastConsolidation > 0) {
                _stakerRewardInfo.earned += _rewardSinceLastConsolidation;
            }
            _stakerRewardInfo.consolidatedPerStakedToken = _reward
                .perStakedToken;
        }
        lastConsolidationTimestamp = _consolidationTimestamp;
    }

    function claimableRewards(address _account)
        public
        view
        returns (uint256[] memory)
    {
        uint256[] memory _outstandingRewards = new uint256[](rewards.length);
        if (!initialized || block.timestamp < startingTimestamp) {
            for (uint256 _i; _i < rewards.length; _i++) {
                _outstandingRewards[_i] = 0;
            }
            return _outstandingRewards;
        }
        Staker storage _staker = stakers[_account];
        uint64 _consolidationTimestamp =
            uint64(Math.min(block.timestamp, endingTimestamp));
        uint256 _lastPeriodDuration =
            uint256(_consolidationTimestamp - lastConsolidationTimestamp);
        for (uint256 _i; _i < rewards.length; _i++) {
            Reward storage _reward = rewards[_i];
            StakerRewardInfo storage _stakerRewardInfo =
                _staker.rewardInfo[_reward.token];
            uint256 _localRewardPerStakedToken = _reward.perStakedToken;
            if (_lastPeriodDuration > 0 && totalStakedTokensAmount > 0) {
                _localRewardPerStakedToken += ((_lastPeriodDuration *
                    _reward.amount *
                    MULTIPLIER) / (totalStakedTokensAmount * secondsDuration));
            }
            uint256 _rewardSinceLastConsolidation =
                (_staker.stake *
                    (_localRewardPerStakedToken -
                        _stakerRewardInfo.consolidatedPerStakedToken)) /
                    MULTIPLIER;
            _outstandingRewards[_i] =
                _rewardSinceLastConsolidation +
                (_stakerRewardInfo.earned - _stakerRewardInfo.claimed);
        }
        return _outstandingRewards;
    }

    function getRewardTokens() external view returns (address[] memory) {
        address[] memory _rewardTokens = new address[](rewards.length);
        for (uint256 _i = 0; _i < rewards.length; _i++) {
            _rewardTokens[_i] = rewards[_i].token;
        }
        return _rewardTokens;
    }

    function rewardAmount(address _rewardToken)
        external
        view
        returns (uint256)
    {
        for (uint256 _i = 0; _i < rewards.length; _i++) {
            Reward storage _reward = rewards[_i];
            if (_rewardToken == _reward.token) return _reward.amount;
        }
        return 0;
    }

    function stakedTokensOf(address _staker) external view returns (uint256) {
        return stakers[_staker].stake;
    }

    function earnedRewardsOf(address _staker)
        external
        view
        returns (uint256[] memory)
    {
        Staker storage _stakerFromStorage = stakers[_staker];
        uint256[] memory _earnedRewards = new uint256[](rewards.length);
        for (uint256 _i; _i < rewards.length; _i++) {
            _earnedRewards[_i] = _stakerFromStorage.rewardInfo[
                rewards[_i].token
            ]
                .earned;
        }
        return _earnedRewards;
    }

    function recoverableUnassignedReward(address _rewardToken)
        external
        view
        returns (uint256)
    {
        for (uint256 _i = 0; _i < rewards.length; _i++) {
            Reward storage _reward = rewards[_i];
            if (_reward.token == _rewardToken) {
                uint256 _nonRequiredFunds =
                    _reward.claimed +
                        ((_reward.recoverableSeconds * _reward.amount) /
                            (uint256(secondsDuration) * MULTIPLIER));
                return
                    IERC20(_reward.token).balanceOf(address(this)) -
                    (_reward.amount - _nonRequiredFunds);
            }
        }
        return 0;
    }

    function getClaimedRewards(address _claimer)
        external
        view
        returns (uint256[] memory)
    {
        Staker storage _staker = stakers[_claimer];
        uint256[] memory _claimedRewards = new uint256[](rewards.length);
        for (uint256 _i = 0; _i < rewards.length; _i++) {
            Reward storage _reward = rewards[_i];
            _claimedRewards[_i] = _staker.rewardInfo[_reward.token].claimed;
        }
        return _claimedRewards;
    }

    function renounceOwnership() public onlyOwner {
        owner = address(0);
        emit OwnershipTransferred(owner, address(0));
    }

    function transferOwnership(address _newOwner) public onlyOwner {
        require(_newOwner != address(0), "SRD16");
        emit OwnershipTransferred(owner, _newOwner);
        owner = _newOwner;
    }

    modifier onlyOwner() {
        require(owner == msg.sender, "SRD17");
        _;
    }

    modifier onlyUninitialized() {
        require(!initialized, "SRD18");
        _;
    }

    modifier onlyStarted() {
        require(
            initialized && !canceled && block.timestamp >= startingTimestamp,
            "SRD20"
        );
        _;
    }

    modifier onlyRunning() {
        require(
            initialized &&
                !canceled &&
                block.timestamp >= startingTimestamp &&
                block.timestamp <= endingTimestamp,
            "SRD21"
        );
        _;
    }
}
```
### **contracts/ERC20StakingRewardsDistributionFactory.sol**
```javascript
// SPDX-License-Identifier: GPL-3.0

pragma solidity ^0.8.0;

import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
import "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";
import "@openzeppelin/contracts/access/Ownable.sol";
import "@openzeppelin/contracts/proxy/Clones.sol";
import "./interfaces/IERC20StakingRewardsDistribution.sol";

/**
 * Errors codes:
 *
 * SRF01: cannot pause staking (already paused)
 * SRF02: cannot resume staking (already active)
 */
contract ERC20StakingRewardsDistributionFactory is Ownable {
    using SafeERC20 for IERC20;

    address public implementation;
    bool public stakingPaused;
    IERC20StakingRewardsDistribution[] public distributions;

    event DistributionCreated(address owner, address deployedAt);

    constructor(address _implementation) {
        implementation = _implementation;
    }

    function upgradeImplementation(address _implementation) external onlyOwner {
        implementation = _implementation;
    }

    function pauseStaking() external onlyOwner {
        require(!stakingPaused, "SRF01");
        stakingPaused = true;
    }

    function resumeStaking() external onlyOwner {
        require(stakingPaused, "SRF02");
        stakingPaused = false;
    }

    function createDistribution(
        address[] calldata _rewardTokenAddresses,
        address _stakableTokenAddress,
        uint256[] calldata _rewardAmounts,
        uint64 _startingTimestamp,
        uint64 _endingTimestamp,
        bool _locked,
        uint256 _stakingCap
    ) public virtual {
        address _distributionProxy = Clones.clone(implementation);
        for (uint256 _i; _i < _rewardTokenAddresses.length; _i++) {
            IERC20(_rewardTokenAddresses[_i]).safeTransferFrom(
                msg.sender,
                _distributionProxy,
                _rewardAmounts[_i]
            );
        }
        IERC20StakingRewardsDistribution _distribution =
            IERC20StakingRewardsDistribution(_distributionProxy);
        _distribution.initialize(
            _rewardTokenAddresses,
            _stakableTokenAddress,
            _rewardAmounts,
            _startingTimestamp,
            _endingTimestamp,
            _locked,
            _stakingCap
        );
        _distribution.transferOwnership(msg.sender);
        distributions.push(_distribution);
        emit DistributionCreated(msg.sender, address(_distribution));
    }

    function getDistributionsAmount() external view returns (uint256) {
        return distributions.length;
    }
}
```


### **After that go to the migrations folder, replace 1_initial_migration.js with 1_deploy_test_dependencies.js and paste the code**
```javascript
const FirstRewardERC20 = artifacts.require("FirstRewardERC20");
const SecondRewardERC20 = artifacts.require("SecondRewardERC20");
const ZeroDecimalsRewardERC20 = artifacts.require("ZeroDecimalsRewardERC20");
const FirstStakableERC20 = artifacts.require("FirstStakableERC20");
const SecondStakableERC20 = artifacts.require("SecondStakableERC20");
const ERC20StakingRewardsDistribution = artifacts.require(
    "ERC20StakingRewardsDistribution"
);
const ERC20StakingRewardsDistributionFactory = artifacts.require(
    "ERC20StakingRewardsDistributionFactory"
);

module.exports = async (deployer, network) => {
    if (network === "soliditycoverage") {
        await deployer.deploy(ERC20StakingRewardsDistribution);
        deployer.deploy(FirstRewardERC20);
        deployer.deploy(SecondRewardERC20);
        deployer.deploy(ZeroDecimalsRewardERC20);
        deployer.deploy(FirstStakableERC20);
        deployer.deploy(SecondStakableERC20);
        deployer.deploy(
            ERC20StakingRewardsDistributionFactory,
            ERC20StakingRewardsDistribution.address
        );
    }
};
```

## **Final step: let's run test the ERC20 Staking Smart Contracts & upload it to our github, we will use this as an important dependencies to excecute the Staking Rewards Program on Frontend**

**Prettier**
```bash
yarn prettier -l contracts/**/*.sol && prettier -l test/**/*.js
```

**Test**
```bash
yarn truffle test --runner-output-only
```

**Test Coverage**
```bash
yarn truffle run coverage
```

**Test Gas Report**
```bash
yarn truffle test --gas-report
```
#### Result Sample
```javascript
 Compiling your contracts...
===========================
✔ Fetching solc version list from solc-bin. Attempt #1
> Everything is up to date, there is nothing to compile.



  Contract: ERC20StakingRewardsDistribution - Multi rewards, single stakable token - Cancelation
    ✓ should succeed in the right conditions (8919ms)
    ✓ shouldn't allow for a second initialization on success (13231ms)
    ✓ shouldn't allow for a second cancelation on success (10489ms)

  Contract: ERC20StakingRewardsDistribution - Single stakable, multi reward tokens - Claiming
    ✓ should succeed in claiming the full reward if only one staker stakes right from the first second (146000ms)
    ✓ should fail when claiming zero rewards (claimAll) (67768ms)
    ✓ should succeed when claiming zero first rewards and all of the second rewards (94924ms)
    ✓ should succeed when claiming zero first reward and all of the second reward (17591ms)
    ✓ should succeed when claiming zero first rewards and part of the second rewards (17404ms)
    ✓ should succeed when claiming zero first reward and all of the second reward (16042ms)
    ✓ should succeed in claiming two multiple rewards if two stakers stake exactly the same amount at different times (28330ms)
    ✓ should succeed in claiming three rewards if three stakers stake exactly the same amount at different times (36022ms)
    ✓ should succeed in claiming a reward if a staker stakes when the distribution has already started (15723ms)
    ✓ should fail in claiming 0 rewards if a staker stakes at the last second (literally) (17122ms)
    ✓ should succeed in claiming one rewards if a staker stakes at the last valid distribution second (15151ms)
    ✓ should succeed in claiming two rewards if two stakers stake exactly the same amount at different times, and then the first staker withdraws a portion of his stake (29908ms)
    ✓ should succeed in claiming two rewards if two stakers both stake at the last valid distribution second (24393ms)
    ✓ should succeed in claiming a reward if a staker stakes at second n and then increases their stake (21770ms)
    ✓ should succeed in claiming two rewards if two staker respectively stake and withdraw at the same second (26220ms)
    ✓ should fail when trying to claim passing an excessive-length amounts array (10427ms)
    ✓ should fail when trying to claim passing a defective-length amounts array (9601ms)
    ✓ should fail when trying to claim only a part of the reward, if the first passed in amount is bigger than allowed (24572ms)
    ✓ should fail when trying to claim only a part of the reward, if the second passed in amount is bigger than allowed (13752ms)
    ✓ should fail when trying to claim only a part of the reward, if the second passed in amount is bigger than allowed (14008ms)
    ✓ should fail when trying to claim only a part of the reward, if the second passed in amount is bigger than allowed (14000ms)
    ✓ should succeed in claiming specific amounts under the right conditions (14733ms)
    ✓ should succeed in claiming specific amounts to a foreign address under the right conditions (17793ms)

  Contract: ERC20StakingRewardsDistribution - Multi rewards, single stakable token - Initialization
    ✓ should fail when reward tokens/amounts arrays have inconsistent lengths (3913ms)
    ✓ should fail when funding for the first reward token has not been sent to the contract before calling initialize (3979ms)
    ✓ should fail when funding for the second reward token has not been sent to the contract before calling initialize (52065ms)
    ✓ should fail when passing a 0-address second reward token (3893ms)
    ✓ should fail when passing 0 as the first reward amount (14181ms)
```



## **2. Create Staking Distributions Contracts**
Simply type this command in your terminal

```bash
mkdir figment-staking
cd figment-staking
yarn add hardhat
npx hardhat 
```

## **File Structure**
```
figment-staking
├── contracts
│   |── interfaces
│   |   ├── IDEXswapERC20StakingRewardsDistributionFactory.sol
│   |   ├── IDEXTokenRegistry.sol 
│   |   ├── IStakableTokenValidator.sol 
|   |   └── IRewardTokensValidator.sol
|   |── test
│   |    |── 5
|   |    |   └── TestDependencies.4.sol
|   |    └── TestDependencies.sol
│   |── DeXsRewardTokensValidator.sol
│   |── DeXsStakableTokenValidator.sol
│   |── DexSwapERC20StakingRewardsDistributionFactory.sol
├── scripts
|      └── flattener.sh
├── tasks
|      └── deploy.js
├── .env
├── .eslintrc
├── .solcover.js
├── commitlint.config.js
├── hardhat.config.js
├── package.json
└────────────────────────
```


### **Add the ERC20 Staking Rewards Contract that we made before**
```bash
yarn add git://github.com/my_github_name/ERC20-Staking-Rewards.git
```


### **Add Token Registry**
```bash
yarn add git://github.com/Agin-DropDisco/dexswap-registry.git 
```


### **Add Core Core Smart Contracts that we made in Part I**
```bash
yarn add git://github.com/my_github_name/dexswap-core.git
```


### **contracts/interfaces/IDEXswapERC20StakingRewardsDistributionFactory.sol**
```javascript
// SPDX-License-Identifier: GPL-3.0

pragma solidity >=0.8.0;

import "erc20-staking-rewards/contracts/interfaces/IERC20StakingRewardsDistributionFactory.sol";

interface IDEXswapERC20StakingRewardsDistributionFactory is
    IERC20StakingRewardsDistributionFactory
{
    function setRewardTokensValidator(address _rewardTokensValidatorAddress)
        external;

    function setStakableTokenValidator(address _stakableTokenValidatorAddress)
        external;
}
```


### **contracts/interfaces/IDEXTokenRegistry.sol**
```javascript
// SPDX-License-Identifier: GPL-3.0

pragma solidity >=0.8.0;

interface IDEXTokenRegistry {
    function isTokenActive(uint256 _listId, address _token)
        external
        view
        returns (bool);
}
```


### **contracts/interfaces/IStakableTokenValidator.sol**
```javascript
// SPDX-License-Identifier: GPL-3.0

pragma solidity >=0.8.0;

interface IStakableTokenValidator {
    function validateToken(address _token) external view;
}
```


### **contracts/interfaces/IRewardTokensValidator.sol**
```javascript
// SPDX-License-Identifier: GPL-3.0

pragma solidity >=0.8.0;

interface IRewardTokensValidator {
    function validateTokens(address[] calldata _tokens) external view;
}
```


### **ontracts/test/5/TestDependencies.4.sol**
```javascript
//SPDX-License-Identifier: GPL-3.0

pragma solidity =0.5.16;

import "dexswap-core/contracts/DexSwapFactory.sol";
import "dexswap-core/contracts/DexSwapPair.sol";

contract FakeDexSwapPair is DexSwapPair {
    constructor(address _token0, address _token1) public {
        token0 = _token0;
        token1 = _token1;
    }
}

contract FailingToken0GetterDexSwapPair {
    address public token1;

    constructor(address _token1) public {
        token1 = _token1;
    }

    function token0() external pure returns (address) {
        revert("failed");
    }
}

contract FailingToken1GetterDexSwapPair {
    address public token0;

    constructor(address _token0) public {
        token0 = _token0;
    }

    function token1() external pure returns (address) {
        revert("failed");
    }
}
```


### **contracts/TestDependencies.sol**
```javascript
//SPDX-License-Identifier: GPL-3.0

pragma solidity ^0.8.4;

import "@openzeppelin/contracts/token/ERC20/presets/ERC20PresetMinterPauser.sol";
import "dexswap-core/contracts/interfaces/IDexSwapFactory.sol";
import "dexswap-token-registry/contracts/DEXTokenRegistry.sol";
import "erc20-staking-rewards/contracts/ERC20StakingRewardsDistribution.sol";

contract FirstRewardERC20 is ERC20PresetMinterPauser {
    constructor() ERC20PresetMinterPauser("First reward", "RWD1") {}
}

contract SecondRewardERC20 is ERC20PresetMinterPauser {
    constructor() ERC20PresetMinterPauser("Second reward", "RWD2") {}
}

contract FirstStakableERC20 is ERC20PresetMinterPauser {
    constructor() ERC20PresetMinterPauser("First stakable", "STK1") {}
}

contract SecondStakableERC20 is ERC20PresetMinterPauser {
    constructor() ERC20PresetMinterPauser("Second stakable", "STK2") {}
}
```


### **contracts/DeXsRewardTokensValidator.sol**
```javascript
// SPDX-License-Identifier: GPL-3.0

pragma solidity ^0.8.4;

import "@openzeppelin/contracts/access/Ownable.sol";
import "./interfaces/IRewardTokensValidator.sol";
import "./interfaces/IDEXTokenRegistry.sol";

contract DefaultRewardTokensValidator is IRewardTokensValidator, Ownable {
    IDEXTokenRegistry public dexTokenRegistry;
    uint256 public dexTokenRegistryListId;

    constructor(address _dexTokenRegistryAddress, uint256 _dexTokenRegistryListId)
    {
        require(
            _dexTokenRegistryAddress != address(0),
            "DefaultRewardTokensValidator: 0-address token registry address"
        );
        require(
            _dexTokenRegistryListId > 0,
            "DefaultRewardTokensValidator: invalid token list id"
        );
        dexTokenRegistry = IDEXTokenRegistry(_dexTokenRegistryAddress);
        dexTokenRegistryListId = _dexTokenRegistryListId;
    }

    function setDexTokenRegistry(address _dexTokenRegistryAddress)
        external
        onlyOwner
    {
        require(
            _dexTokenRegistryAddress != address(0),
            "DefaultRewardTokensValidator: 0-address token registry address"
        );
        dexTokenRegistry = IDEXTokenRegistry(_dexTokenRegistryAddress);
    }

    function setDexTokenRegistryListId(uint256 _dexTokenRegistryListId)
        external
        onlyOwner
    {
        require(
            _dexTokenRegistryListId > 0,
            "DefaultRewardTokensValidator: invalid token list id"
        );
        dexTokenRegistryListId = _dexTokenRegistryListId;
    }

    function validateTokens(address[] calldata _rewardTokens)
        external
        view
        override
    {
        require(
            _rewardTokens.length > 0,
            "DefaultRewardTokensValidator: 0-length reward tokens array"
        );
        for (uint256 _i = 0; _i < _rewardTokens.length; _i++) {
            address _rewardToken = _rewardTokens[_i];
            require(
                _rewardToken != address(0),
                "DefaultRewardTokensValidator: 0-address reward token"
            );
            require(
                dexTokenRegistry.isTokenActive(
                    dexTokenRegistryListId,
                    _rewardToken
                ),
                "DefaultRewardTokensValidator: invalid reward token"
            );
        }
    }
}
```


### **contracts/DeXsStakableTokenValidator.sol**
```javascript
// SPDX-License-Identifier: GPL-3.0

pragma solidity ^0.8.4;

import "./interfaces/IStakableTokenValidator.sol";
import "./interfaces/IDEXTokenRegistry.sol";
import "dexswap-core/contracts/interfaces/IDexSwapPair.sol";
import "dexswap-core/contracts/interfaces/IDexSwapFactory.sol";
import "@openzeppelin/contracts/access/Ownable.sol";

contract DefaultStakableTokenValidator is IStakableTokenValidator, Ownable {
    IDEXTokenRegistry public dexTokenRegistry;
    uint256 public dexTokenRegistryListId;
    IDexSwapFactory public dexSwapFactory;

    constructor(
        address _dexTokenRegistryAddress,
        uint256 _dexTokenRegistryListId,
        address _dexSwapFactoryAddress
    ) {
        require(
            _dexTokenRegistryAddress != address(0),
            "DefaultStakableTokenValidator: 0-address token registry address"
        );
        require(
            _dexTokenRegistryListId > 0,
            "DefaultStakableTokenValidator: invalid token list id"
        );
        require(
            _dexSwapFactoryAddress != address(0),
            "DefaultStakableTokenValidator: 0-address factory address"
        );
        dexTokenRegistry = IDEXTokenRegistry(_dexTokenRegistryAddress);
        dexTokenRegistryListId = _dexTokenRegistryListId;
        dexSwapFactory = IDexSwapFactory(_dexSwapFactoryAddress);
    }

    function setDexTokenRegistry(address _dexTokenRegistryAddress)
        external
        onlyOwner
    {
        require(
            _dexTokenRegistryAddress != address(0),
            "DefaultStakableTokenValidator: 0-address token registry address"
        );
        dexTokenRegistry = IDEXTokenRegistry(_dexTokenRegistryAddress);
    }

    function setDexTokenRegistryListId(uint256 _dexTokenRegistryListId)
        external
        onlyOwner
    {
        require(
            _dexTokenRegistryListId > 0,
            "DefaultStakableTokenValidator: invalid token list id"
        );
        dexTokenRegistryListId = _dexTokenRegistryListId;
    }

    function setDexSwapFactory(address _dexSwapFactoryAddress)
        external
        onlyOwner
    {
        require(
            _dexSwapFactoryAddress != address(0),
            "DefaultStakableTokenValidator: 0-address factory address"
        );
        dexSwapFactory = IDexSwapFactory(_dexSwapFactoryAddress);
    }

    function validateToken(address _stakableTokenAddress)
        external
        view
        override
    {
        require(
            _stakableTokenAddress != address(0),
            "DefaultStakableTokenValidator: 0-address stakable token"
        );
        IDexSwapPair _potentialDexSwapPair = IDexSwapPair(_stakableTokenAddress);
        address _token0;
        try _potentialDexSwapPair.token0() returns (address _fetchedToken0) {
            _token0 = _fetchedToken0;
        } catch {
            revert(
                "DefaultStakableTokenValidator: could not get token0 for pair"
            );
        }
        require(
            dexTokenRegistry.isTokenActive(dexTokenRegistryListId, _token0),
            "DefaultStakableTokenValidator: invalid token 0 in DeXs pair"
        );
        address _token1;
        try _potentialDexSwapPair.token1() returns (address _fetchedToken1) {
            _token1 = _fetchedToken1;
        } catch {
            revert(
                "DefaultStakableTokenValidator: could not get token1 for pair"
            );
        }
        require(
            dexTokenRegistry.isTokenActive(dexTokenRegistryListId, _token1),
            "DefaultStakableTokenValidator: invalid token 1 in DeXs pair"
        );
        require(
            dexSwapFactory.getPair(_token0, _token1) == _stakableTokenAddress,
            "DefaultStakableTokenValidator: pair not registered in factory"
        );
    }
}
```


### **contracts/DexSwapERC20StakingRewardsDistributionFactory.sol**
```javascript
// SPDX-License-Identifier: GPL-3.0

pragma solidity ^0.8.4;

import "erc20-staking-rewards/contracts/ERC20StakingRewardsDistributionFactory.sol";
import "./interfaces/IRewardTokensValidator.sol";
import "./interfaces/IStakableTokenValidator.sol";

contract DexSwapERC20StakingRewardsDistributionFactory is
    ERC20StakingRewardsDistributionFactory
{
    IRewardTokensValidator public rewardTokensValidator;
    IStakableTokenValidator public stakableTokenValidator;

    constructor(
        address _rewardTokensValidatorAddress,
        address _stakableTokenValidatorAddress,
        address _implementation
    ) ERC20StakingRewardsDistributionFactory(_implementation) {
        rewardTokensValidator = IRewardTokensValidator(
            _rewardTokensValidatorAddress
        );
        stakableTokenValidator = IStakableTokenValidator(
            _stakableTokenValidatorAddress
        );
    }

    function setRewardTokensValidator(address _rewardTokensValidatorAddress)
        external
        onlyOwner
    {
        rewardTokensValidator = IRewardTokensValidator(
            _rewardTokensValidatorAddress
        );
    }

    function setStakableTokenValidator(address _stakableTokenValidatorAddress)
        external
        onlyOwner
    {
        stakableTokenValidator = IStakableTokenValidator(
            _stakableTokenValidatorAddress
        );
    }

    function createDistribution(
        address[] calldata _rewardTokensAddresses,
        address _stakableTokenAddress,
        uint256[] calldata _rewardAmounts,
        uint64 _startingTimestamp,
        uint64 _endingTimestmp,
        bool _locked,
        uint256 _stakingCap
    ) public override {
        if (address(rewardTokensValidator) != address(0)) {
            rewardTokensValidator.validateTokens(_rewardTokensAddresses);
        }
        if (address(stakableTokenValidator) != address(0)) {
            stakableTokenValidator.validateToken(_stakableTokenAddress);
        }
        ERC20StakingRewardsDistributionFactory.createDistribution(
            _rewardTokensAddresses,
            _stakableTokenAddress,
            _rewardAmounts,
            _startingTimestamp,
            _endingTimestmp,
            _locked,
            _stakingCap
        );
    }
}
```


### **flattener.sh**
```javascript
npx truffle-flattener contracts/DexSwapERC20StakingRewardsDistributionFactory.sol > contracts/.flattened/DexSwapERC20StakingRewardsDistributionFactory.sol
```


### **task/deploy.js**
```javascript
const { task } = require("hardhat/config");

task(
    "deploy",
    "Deploys the whole contracts suite and verifies source code on Etherscan"
)
    .addFlag("withValidators")
    .addOptionalParam("tokenRegistryAddress", "The token registry address")
    .addOptionalParam(
        "tokenRegistryListId",
        "The token registry list id to be used to validate tokens"
    )
    .addOptionalParam("factoryAddress", "The address of DexSwap's pairs factory")
    .addOptionalParam("ownerAddress", "The address of the owner")
    .addFlag(
        "verify",
        "Additional (and optional) Etherscan contracts verification"
    )
    .setAction(async (taskArguments, hre) => {
        const {
            tokenRegistryAddress,
            tokenRegistryListId,
            factoryAddress,
            verify,
            withValidators,
            ownerAddress,
        } = taskArguments;

        if (
            withValidators &&
            (!tokenRegistryAddress || !tokenRegistryListId || !factoryAddress)
        ) {
            throw new Error(
                "token registry address/list di and factory address are required"
            );
        }

        await hre.run("clean");
        await hre.run("compile");

        let rewardTokensValidator = {
            address: "0x0000000000000000000000000000000000000000",
        };
        let stakableTokenValidator = {
            address: "0x0000000000000000000000000000000000000000",
        };
        if (withValidators) {
            const DefaultRewardTokensValidator = hre.artifacts.require(
                "DefaultRewardTokensValidator"
            );
            rewardTokensValidator = await DefaultRewardTokensValidator.new(
                tokenRegistryAddress,
                tokenRegistryListId
            );

            const DefaultStakableTokenValidator = hre.artifacts.require(
                "DefaultStakableTokenValidator"
            );
            stakableTokenValidator = await DefaultStakableTokenValidator.new(
                tokenRegistryAddress,
                tokenRegistryListId,
                factoryAddress
            );
        }

        const ERC20StakingRewardsDistribution = hre.artifacts.require(
            "ERC20StakingRewardsDistribution"
        );
        const erc20DistributionImplementation = await ERC20StakingRewardsDistribution.new();
        const DexSwapERC20StakingRewardsDistributionFactory = hre.artifacts.require(
            "DexSwapERC20StakingRewardsDistributionFactory"
        );
        const factory = await DexSwapERC20StakingRewardsDistributionFactory.new(
            rewardTokensValidator.address,
            stakableTokenValidator.address,
            erc20DistributionImplementation.address
        );

        
        if (ownerAddress) {
            await factory.transferOwnership(ownerAddress);
            console.log(`ownership transferred to ${ownerAddress}`);
        }

        if (verify) {
            await new Promise((resolve) => {
                console.log("waiting");
                setTimeout(resolve, 60000);
            });
            if (withValidators) {
                await hre.run("verify", {
                    address: rewardTokensValidator.address,
                    constructorArgsParams: [
                        tokenRegistryAddress,
                        tokenRegistryListId,
                    ],
                });
                await hre.run("verify", {
                    address: stakableTokenValidator.address,
                    constructorArgsParams: [
                        tokenRegistryAddress,
                        tokenRegistryListId,
                        factoryAddress,
                    ],
                });
            }
            await hre.run("verify", {
                address: erc20DistributionImplementation.address,
                constructorArgsParams: [],
            });
            await hre.run("verify", {
                address: factory.address,
                constructorArgsParams: [
                    rewardTokensValidator.address,
                    stakableTokenValidator.address,
                    erc20DistributionImplementation.address,
                ],
            });
            console.log(`source code verified`);
        }

        if (withValidators) {
            console.log(
                `reward tokens validator deployed at address ${rewardTokensValidator.address}`
            );
            console.log(
                `stakable token validator deployed at address ${stakableTokenValidator.address}`
            );
        }
        console.log(`factory deployed at address ${factory.address}`);
    });
```


### **.env**
```javascript
MNEMONIC=
INFURA_ID=
PRIVATE_KEY=
ETHERSCAN_API_KEY=//if needed eg: rinkeby etc
```


### **.solcover.js**
```javascript
module.exports = {
    skipFiles: [
        "/test/TestDependencies.sol",
        "/abstraction/IRewardTokensValidator.sol",
        "/abstraction/IStakableTokensValidator.sol",
    ],
};
```


### **hardhat.config.js**
```javascript
require("dotenv").config();
require("@nomiclabs/hardhat-truffle5");
require("solidity-coverage");
require("hardhat-gas-reporter");
require("@nomiclabs/hardhat-etherscan");
require("./tasks/deploy");

const infuraId = process.env.INFURA_ID;

/**
 * @type import('hardhat/config').HardhatUserConfig
 */
module.exports = {
    networks: {
        mainnet: {
            url: `https://mainnet.infura.io/v3/${infuraId}`,
        },
        matic: {
            url:"https://rpc-mainnet.matic.quiknode.pro",
            accounts: [process.env.PRIVATE_KEY],
            network_id: 137,
            gasPrice: 2100000000,
            timeout: 100000
        },
        mumbai: {
            url:"https://matic-mumbai.chainstacklabs.com",
            accounts: [process.env.PRIVATE_KEY],
            network_id: 80001,
            gasPrice: 2100000000,
            timeout: 100000
        }
    },
    solidity: {
        compilers: [
            {
                version: "0.8.4",
                settings: {
                    optimizer: {
                        enabled: true,
                        runs: 200,
                    },
                },
            },
            {
                version: "0.5.16",
                settings: {
                    optimizer: {
                        enabled: true,
                        runs: 200,
                    },
                },
            },
        ],
    },
    gasReporter: {
        currency: "USD",
        enabled: process.env.GAS_REPORT_ENABLED === "true",
    },
    etherscan: {
        apiKey: process.env.ETHERSCAN_API_KEY,
    },
};
```


### **Paste to Package.json**
```json
    "scripts": {
        "lint:eslint": "eslint \"test/**/*.js\"",
        "lint:prettier": "prettier -l contracts/**/*.sol && prettier -l test/**/*.js",
        "lint:commit-message": "commitlint -e",
        "lint": "yarn lint:eslint && yarn lint:prettier",
        "test": "hardhat test",
        "test:coverage": "hardhat coverage",
        "test:gasreport": "cross-env GAS_REPORT_ENABLED=true hardhat test",
        "compile": "rimraf ./build && rimraf ./artifacts && hardhat compile && mkdirp build && copyfiles -f -E ./artifacts/contracts/*.sol/*.json ./build && rimraf ./build/*.dbg.json",
        "prepack": "yarn compile && copyfiles -f -E ./contracts/*.sol ./ && copyfiles -f -E ./contracts/interfaces/*.sol ./interfaces",
        "postpack": "rimraf ./*.sol",
        "deploy:matic:no-validators": "hardhat deploy --network matic --owner-address 0x123456789..... --factory-address 0x123456789.....",
        "deploy:mumbai:no-validators": "hardhat deploy --network mumbai --owner-address 0x123456789..... --factory-address 0x123456789....."
    },
    "devDependencies": {
        "@commitlint/cli": "^11.0.0",
        "@commitlint/config-conventional": "^11.0.0",
        "@nomiclabs/hardhat-etherscan": "^2.1.1",
        "@nomiclabs/hardhat-truffle5": "^2.0.0",
        "@nomiclabs/hardhat-web3": "^2.0.0",
        "babel-eslint": "^10.1.0",
        "bn.js": "^5.1.3",
        "flattener": "^0.4.0",
        "chai": "^4.2.0",
        "copyfiles": "^2.4.1",
        "cross-env": "^7.0.3",
        "dotenv": "^8.2.0",
        "eslint": "^7.17.0",
        "hardhat": "^2.0.7",
        "hardhat-gas-reporter": "^1.0.4",
        "husky": "^4.3.7",
        "mkdirp": "^1.0.4",
        "prettier": "^2.1.2",
        "prettier-plugin-solidity": "^1.0.0-beta.3",
        "rimraf": "^3.0.2",
        "solhint-plugin-prettier": "^0.0.5",
        "solidity-coverage": "^0.7.13",
        "web3": "^1.3.1"
    }
```


### **Type this command in your terminal**
```bash
yarn i && yarn compile && yarn deploy:matic:no-validators
```

