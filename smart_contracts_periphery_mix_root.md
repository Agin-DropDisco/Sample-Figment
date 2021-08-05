## Part II.
### Clone, Edit & Deploy Smart Contracts Periphery
**Workflow**
<img src="https://gateway.pinata.cloud/ipfs/QmVtjSZNMsJSuk9ai7jyrixhmfoWiV8CEd1PVr6vvk9jMV" align="center">

```javascript
git clone https://github.com/Agin-DropDisco/figment-periphery.git
cd figment-periphery && yarn i
```
#### Create New Javascript File
**2_deploy.js**
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




## Part III.
Create/Deploy Staking Smart Contracts using Truffle & Hardhat
- Workflow
<img src="https://gateway.pinata.cloud/ipfs/QmUGzgoxJyWaff4g2hjBnaiKUHvwqnNMWwY2prEm8tr6ez" align="center">

1. Create ERC20 Staking Rewards Distributions Contracts
2. Create Staking Distributions Contracts

## Create ERC20 Staking Rewards Distributions Contracts
A generic contracts suite to bootstrap staking campaigns in which stakers get distributed rewards over time in relation to their share of the total staked tokens. Supports multiple ERC20 reward tokens, locked campaigns (i.e. no withdrawals until the end of the distribution if tokens are staked), capped campaigns, and rewards recovery by the owner for those dead moments in which no tokens are staked.(see User Interaction Workflow)
### Installation {#installation}
```bash
mkdir ERC20-Staking-Rewards
cd ERC20-Staking-Rewards
truffle init
yarn init
```
Copy & Paste to Package.json
```json
"devDependencies": {
    "@openzeppelin/contracts": "^4.0.0",
    "@commitlint/cli": "^11.0.0",
    "@commitlint/config-conventional": "^11.0.0",
    "babel-eslint": "^10.1.0",
    "bn.js": "^5.1.3",
    "chai": "^4.2.0",
    "eslint": "^7.13.0",
    "eth-gas-reporter": "^0.2.20",
    "ganache-cli": "^6.12.1",
    "husky": "^4.3.0",
    "prettier": "^2.1.2",
    "prettier-plugin-solidity": "^1.0.0-beta.2",
    "solhint-plugin-prettier": "^0.0.5",
    "solidity-coverage": "^0.7.13",
    "truffle": "^5.1.58"
}
"scripts": {
    "lint:eslint": "eslint \"test/**/*.js\"",
    "lint:prettier": "prettier -l contracts/**/*.sol && prettier -l test/**/*.js",
    "lint:commit-message": "commitlint -e",
    "lint": "yarn lint:eslint && yarn lint:prettier",
    "test": "truffle test --runner-output-only",
    "test:coverage": "truffle run coverage",
    "test:gasreport": "truffle test --gas-report",
    "compile": "truffle compile",
    "prepack": "cp ./contracts/*.sol ./ && mkdir ./interfaces && cp ./contracts/interfaces/*.sol ./interfaces",
    "postpack": "rm -rf ./*.sol rm -rf ./interfaces"
    }
```
```bash
yarn i
```
```bash
cd contracts
mkdir interfaces
cd interfaces
```
### Create New Solidity File
#### IERC20StakingRewardsDistribution.sol
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
#### IERC20StakingRewardsDistributionFactory.sol
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

```bash
cd ..
```


#### ERC20StakingRewardsDistribution.sol
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
#### ERC20StakingRewardsDistributionFactory.sol
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


**go to migrations folder, replace 1_initial_migration.js with 1_deploy_test_dependencies.js and paste the code**
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
Test Dependencies
```bash
truffle compile && truffle migrate --network soliditycoverage
```

**Let's Create Test script**
#### Inside of test folder create new folder & file
**Folder Structure**
```
test
├── constants
│   └── index.js
├── erc20-distribution
│   |── multi-rewards-single-stakable
│   |   ├── cancelation.test.js
│   |   ├── claiming.test.js
│   |   ├── initialization.test.js
│   |   ├── recovering.test.js
│   |   ├── cancelation.test.js
|   |   |
│   |── single-tokens
│   |   ├── cancelation.test.js
│   |   ├── claimable-rewards.test.js
│   |   ├── claiming.test.js
│   |   ├── initializations.test.js
│   |   ├── recovering.test.js
│   |   ├── staking.test.js
│   |   ├── withdrawing.test.js
|   |   |
├── erc20-distribution-factory
│   |── multi-rewards-single-stakable
│   |   ├── create-distributions.test.js
|   |   |
│   |── single-tokens
│   |   ├── create-distributions.test.js
│   |   ├── pause-staking.test.js
│   |   ├── resume-staking.test.js
├── utils
│   |   ├── assertion.js
│   |   ├── conversion.js
│   |   ├── index.js
│   |   ├── network.js
└────────────────────────────
```
#### test/constants/index.js
```javascript
const BN = require("bn.js");

// Maximum variance allowed between expected values and actual ones.
// Mainly to account for division between integers, and associated rounding.
exports.MAXIMUM_VARIANCE = new BN(1); // 1 wei
exports.ZERO_BN = new BN(0);
exports.ZERO_ADDRESS = "0x0000000000000000000000000000000000000000";
```
#### test/erc20-distribution/multi-rewards-single-stakable/cancelation.test.js
```javascript
const BN = require("bn.js");
const { expect } = require("chai");
const { ZERO_BN } = require("../../constants");
const { initializeDistribution } = require("../../utils");
const { toWei } = require("../../utils/conversion");
const { getEvmTimestamp } = require("../../utils/network");

const ERC20StakingRewardsDistribution = artifacts.require(
    "ERC20StakingRewardsDistribution"
);
const ERC20StakingRewardsDistributionFactory = artifacts.require(
    "ERC20StakingRewardsDistributionFactory"
);
const FirstRewardERC20 = artifacts.require("FirstRewardERC20");
const SecondRewardERC20 = artifacts.require("SecondRewardERC20");
const FirstStakableERC20 = artifacts.require("FirstStakableERC20");

contract(
    "ERC20StakingRewardsDistribution - Multi rewards, single stakable token - Cancelation",
    () => {
        let erc20DistributionFactoryInstance,
            firstRewardsTokenInstance,
            secondRewardsTokenInstance,
            stakableTokenInstance,
            ownerAddress;

        beforeEach(async () => {
            const accounts = await web3.eth.getAccounts();
            ownerAddress = accounts[0];
            const erc20DistributionInstance = await ERC20StakingRewardsDistribution.new(
                { from: ownerAddress }
            );
            erc20DistributionFactoryInstance = await ERC20StakingRewardsDistributionFactory.new(
                erc20DistributionInstance.address,
                { from: ownerAddress }
            );
            firstRewardsTokenInstance = await FirstRewardERC20.new();
            secondRewardsTokenInstance = await SecondRewardERC20.new();
            stakableTokenInstance = await FirstStakableERC20.new();
        });

        it("should succeed in the right conditions", async () => {
            const rewardAmounts = [
                await toWei(10, firstRewardsTokenInstance),
                await toWei(100, secondRewardsTokenInstance),
            ];
            const rewardTokens = [
                firstRewardsTokenInstance,
                secondRewardsTokenInstance,
            ];
            // if not specified, the distribution starts 10 seconds from
            // now, so we have the time to cancel it
            const { erc20DistributionInstance } = await initializeDistribution({
                from: ownerAddress,
                erc20DistributionFactoryInstance,
                stakableToken: stakableTokenInstance,
                rewardTokens,
                rewardAmounts,
                duration: 2,
            });
            await erc20DistributionInstance.cancel({ from: ownerAddress });
            for (let i = 0; i < rewardTokens.length; i++) {
                const rewardToken = rewardTokens[i];
                const rewardAmount = rewardAmounts[i];
                expect(
                    await rewardToken.balanceOf(
                        erc20DistributionInstance.address
                    )
                ).to.be.equalBn(ZERO_BN);
                expect(await rewardToken.balanceOf(ownerAddress)).to.be.equalBn(
                    rewardAmount
                );
            }
            expect(await erc20DistributionInstance.initialized()).to.be.true;
            expect(await erc20DistributionInstance.canceled()).to.be.true;
        });

        it("shouldn't allow for a second initialization on success", async () => {
            const rewardAmounts = [
                await toWei(10, firstRewardsTokenInstance),
                await toWei(10, firstRewardsTokenInstance),
            ];
            const rewardTokens = [
                firstRewardsTokenInstance,
                secondRewardsTokenInstance,
            ];
            // if not specified, the distribution starts 10 seconds from
            // now, so we have the time to cancel it
            const { erc20DistributionInstance } = await initializeDistribution({
                from: ownerAddress,
                erc20DistributionFactoryInstance,
                stakableToken: stakableTokenInstance,
                rewardTokens,
                rewardAmounts,
                duration: 2,
            });
            await erc20DistributionInstance.cancel({ from: ownerAddress });
            try {
                const currentTimestamp = await getEvmTimestamp();
                await erc20DistributionInstance.initialize(
                    rewardTokens.map((token) => token.address),
                    stakableTokenInstance.address,
                    rewardAmounts,
                    currentTimestamp.add(new BN(10)),
                    currentTimestamp.add(new BN(20)),
                    false,
                    0
                );
                throw new Error("should have failed");
            } catch (error) {
                expect(error.message).to.contain("SRD18");
            }
        });

        it("shouldn't allow for a second cancelation on success", async () => {
            const rewardAmounts = [
                await toWei(10, firstRewardsTokenInstance),
                await toWei(10, firstRewardsTokenInstance),
            ];
            const rewardTokens = [
                firstRewardsTokenInstance,
                secondRewardsTokenInstance,
            ];
            // if not specified, the distribution starts 10 seconds from
            // now, so we have the time to cancel it
            const { erc20DistributionInstance } = await initializeDistribution({
                from: ownerAddress,
                erc20DistributionFactoryInstance,
                stakableToken: stakableTokenInstance,
                rewardTokens,
                rewardAmounts,
                duration: 2,
            });
            await erc20DistributionInstance.cancel({ from: ownerAddress });
            try {
                await erc20DistributionInstance.cancel({ from: ownerAddress });
                throw new Error("should have failed");
            } catch (error) {
                expect(error.message).to.contain("SRD19");
            }
        });
    }
);
```
#### test/erc20-distribution/multi-rewards-single-stakable/claiming.test.js
```javascript
const BN = require("bn.js");
const { expect } = require("chai");
const { MAXIMUM_VARIANCE, ZERO_BN } = require("../../constants");
const {
    initializeDistribution,
    initializeStaker,
    stakeAtTimestamp,
    withdrawAtTimestamp,
} = require("../../utils");
const { toWei } = require("../../utils/conversion");
const {
    stopMining,
    startMining,
    fastForwardTo,
    getEvmTimestamp,
} = require("../../utils/network");

const ERC20StakingRewardsDistribution = artifacts.require(
    "ERC20StakingRewardsDistribution"
);
const ERC20StakingRewardsDistributionFactory = artifacts.require(
    "ERC20StakingRewardsDistributionFactory"
);
const FirstRewardERC20 = artifacts.require("FirstRewardERC20");
const SecondRewardERC20 = artifacts.require("SecondRewardERC20");
const FirstStakableERC20 = artifacts.require("FirstStakableERC20");

contract(
    "ERC20StakingRewardsDistribution - Single stakable, multi reward tokens - Claiming",
    () => {
        let erc20DistributionFactoryInstance,
            firstRewardTokenInstance,
            secondRewardTokenInstance,
            stakableTokenInstance,
            ownerAddress,
            firstStakerAddress,
            secondStakerAddress,
            thirdStakerAddress;

        beforeEach(async () => {
            const accounts = await web3.eth.getAccounts();
            ownerAddress = accounts[0];
            const erc20DistributionInstance = await ERC20StakingRewardsDistribution.new(
                { from: ownerAddress }
            );
            erc20DistributionFactoryInstance = await ERC20StakingRewardsDistributionFactory.new(
                erc20DistributionInstance.address,
                { from: ownerAddress }
            );
            firstRewardTokenInstance = await FirstRewardERC20.new();
            secondRewardTokenInstance = await SecondRewardERC20.new();
            stakableTokenInstance = await FirstStakableERC20.new();
            firstStakerAddress = accounts[1];
            secondStakerAddress = accounts[2];
            thirdStakerAddress = accounts[3];
        });

        it("should succeed in claiming the full reward if only one staker stakes right from the first second", async () => {
            const stakedAmount = await toWei(20, stakableTokenInstance);
            const firstRewardAmount = await toWei(10, firstRewardTokenInstance);
            const secondRewardAmount = await toWei(
                20,
                secondRewardTokenInstance
            );
            const {
                erc20DistributionInstance,
                startingTimestamp,
                endingTimestamp,
            } = await initializeDistribution({
                from: ownerAddress,
                erc20DistributionFactoryInstance,
                stakableToken: stakableTokenInstance,
                rewardTokens: [
                    firstRewardTokenInstance,
                    secondRewardTokenInstance,
                ],
                rewardAmounts: [firstRewardAmount, secondRewardAmount],
                duration: 10,
            });
            await initializeStaker({
                erc20DistributionInstance,
                stakableTokenInstance,
                stakerAddress: firstStakerAddress,
                stakableAmount: stakedAmount,
            });
            await fastForwardTo({
                timestamp: startingTimestamp,
            });
            await stakeAtTimestamp(
                erc20DistributionInstance,
                firstStakerAddress,
                stakedAmount,
                startingTimestamp
            );
            const stakerStartingTimestamp = await getEvmTimestamp();
            expect(stakerStartingTimestamp).to.be.equalBn(startingTimestamp);
            // make sure the distribution has ended
            await fastForwardTo({ timestamp: endingTimestamp.add(new BN(1)) });
            await erc20DistributionInstance.claimAll(firstStakerAddress, {
                from: firstStakerAddress,
            });
            const onchainStartingTimestamp = await erc20DistributionInstance.startingTimestamp();
            const onchainEndingTimestamp = await erc20DistributionInstance.endingTimestamp();
            expect(onchainStartingTimestamp).to.be.equalBn(startingTimestamp);
            expect(onchainEndingTimestamp).to.be.equalBn(endingTimestamp);
            const stakingDuration = onchainEndingTimestamp.sub(
                onchainStartingTimestamp
            );

            expect(stakingDuration).to.be.equalBn(new BN(10));
            const firstStakerRewardsTokenBalance = await firstRewardTokenInstance.balanceOf(
                firstStakerAddress
            );
            expect(firstStakerRewardsTokenBalance).to.equalBn(
                firstRewardAmount
            );
            // additional checks to be extra safe
            expect(firstStakerRewardsTokenBalance).to.equalBn(
                firstRewardAmount
            );

            const secondStakerRewardsTokenBalance = await secondRewardTokenInstance.balanceOf(
                firstStakerAddress
            );
            expect(secondStakerRewardsTokenBalance).to.equalBn(
                secondRewardAmount
            );
            // additional checks to be extra safe
            expect(secondStakerRewardsTokenBalance).to.equalBn(
                secondRewardAmount
            );
        });

        it("should fail when claiming zero rewards (claimAll)", async () => {
            const stakedAmount = await toWei(20, stakableTokenInstance);
            const firstRewardsAmount = await toWei(
                10,
                firstRewardTokenInstance
            );
            const secondRewardsAmount = await toWei(
                20,
                secondRewardTokenInstance
            );
            const {
                startingTimestamp,
                erc20DistributionInstance,
            } = await initializeDistribution({
                from: ownerAddress,
                erc20DistributionFactoryInstance,
                stakableToken: stakableTokenInstance,
                rewardTokens: [
                    firstRewardTokenInstance,
                    secondRewardTokenInstance,
                ],
                rewardAmounts: [firstRewardsAmount, secondRewardsAmount],
                duration: 10,
            });
            await initializeStaker({
                erc20DistributionInstance,
                stakableTokenInstance,
                stakerAddress: firstStakerAddress,
                stakableAmount: stakedAmount,
            });
            await fastForwardTo({ timestamp: startingTimestamp });
            try {
                await erc20DistributionInstance.claimAll(firstStakerAddress, {
                    from: firstStakerAddress,
                });
                throw new Error("should have failed");
            } catch (error) {
                expect(error.message).to.contain("SRD23");
            }
        });

        it("should succeed when claiming zero first rewards and all of the second rewards", async () => {
            const stakedAmount = await toWei(20, stakableTokenInstance);
            const firstRewardsAmount = await toWei(
                10,
                firstRewardTokenInstance
            );
            const secondRewardsAmount = await toWei(
                20,
                secondRewardTokenInstance
            );
            const {
                erc20DistributionInstance,
                startingTimestamp,
                endingTimestamp,
            } = await initializeDistribution({
                from: ownerAddress,
                erc20DistributionFactoryInstance,
                stakableToken: stakableTokenInstance,
                rewardTokens: [
                    firstRewardTokenInstance,
                    secondRewardTokenInstance,
                ],
                rewardAmounts: [firstRewardsAmount, secondRewardsAmount],
                duration: 10,
            });
            await initializeStaker({
                erc20DistributionInstance,
                stakableTokenInstance,
                stakerAddress: firstStakerAddress,
                stakableAmount: stakedAmount,
            });
            await fastForwardTo({ timestamp: startingTimestamp });
            // make sure the staking operation happens as soon as possible
            await stakeAtTimestamp(
                erc20DistributionInstance,
                firstStakerAddress,
                stakedAmount,
                startingTimestamp
            );
            await fastForwardTo({ timestamp: endingTimestamp.add(new BN(1)) });
            // staker staked for all of the campaign's duration
            await erc20DistributionInstance.claim(
                [firstRewardsAmount, 0],
                firstStakerAddress,
                { from: firstStakerAddress }
            );
            expect(
                await firstRewardTokenInstance.balanceOf(firstStakerAddress)
            ).to.be.equalBn(firstRewardsAmount);
            expect(
                await secondRewardTokenInstance.balanceOf(firstStakerAddress)
            ).to.be.equalBn(ZERO_BN);
            expect(
                await firstRewardTokenInstance.balanceOf(
                    erc20DistributionInstance.address
                )
            ).to.be.equalBn(ZERO_BN);
            expect(
                await secondRewardTokenInstance.balanceOf(
                    erc20DistributionInstance.address
                )
            ).to.be.equalBn(secondRewardsAmount);
        });

        it("should succeed when claiming zero first reward and all of the second reward", async () => {
            const stakedAmount = await toWei(20, stakableTokenInstance);
            const firstRewardsAmount = await toWei(
                10,
                firstRewardTokenInstance
            );
            const secondRewardsAmount = await toWei(
                20,
                secondRewardTokenInstance
            );
            const {
                startingTimestamp,
                endingTimestamp,
                erc20DistributionInstance,
            } = await initializeDistribution({
                from: ownerAddress,
                erc20DistributionFactoryInstance,
                stakableToken: stakableTokenInstance,
                rewardTokens: [
                    firstRewardTokenInstance,
                    secondRewardTokenInstance,
                ],
                rewardAmounts: [firstRewardsAmount, secondRewardsAmount],
                duration: 10,
            });
            await initializeStaker({
                erc20DistributionInstance,
                stakableTokenInstance,
                stakerAddress: firstStakerAddress,
                stakableAmount: stakedAmount,
            });
            await fastForwardTo({ timestamp: startingTimestamp });
            // make sure the staking operation happens as soon as possible
            await stakeAtTimestamp(
                erc20DistributionInstance,
                firstStakerAddress,
                stakedAmount,
                startingTimestamp
            );
            await fastForwardTo({ timestamp: endingTimestamp.add(new BN(1)) });
            // staker staked for all of the campaign's duration
            await erc20DistributionInstance.claim(
                [0, secondRewardsAmount],
                firstStakerAddress,
                { from: firstStakerAddress }
            );
            expect(
                await firstRewardTokenInstance.balanceOf(firstStakerAddress)
            ).to.be.equalBn(ZERO_BN);
            expect(
                await secondRewardTokenInstance.balanceOf(firstStakerAddress)
            ).to.be.equalBn(secondRewardsAmount);
            expect(
                await firstRewardTokenInstance.balanceOf(
                    erc20DistributionInstance.address
                )
            ).to.be.equalBn(firstRewardsAmount);
            expect(
                await secondRewardTokenInstance.balanceOf(
                    erc20DistributionInstance.address
                )
            ).to.be.equalBn(ZERO_BN);
        });

        it("should succeed when claiming zero first rewards and part of the second rewards", async () => {
            const stakedAmount = await toWei(20, stakableTokenInstance);
            const firstRewardsAmount = await toWei(
                10,
                firstRewardTokenInstance
            );
            const secondRewardsAmount = await toWei(
                20,
                secondRewardTokenInstance
            );
            const {
                erc20DistributionInstance,
                startingTimestamp,
                endingTimestamp,
            } = await initializeDistribution({
                from: ownerAddress,
                erc20DistributionFactoryInstance,
                stakableToken: stakableTokenInstance,
                rewardTokens: [
                    firstRewardTokenInstance,
                    secondRewardTokenInstance,
                ],
                rewardAmounts: [firstRewardsAmount, secondRewardsAmount],
                duration: 10,
            });
            await initializeStaker({
                erc20DistributionInstance,
                stakableTokenInstance,
                stakerAddress: firstStakerAddress,
                stakableAmount: stakedAmount,
            });
            await fastForwardTo({ timestamp: startingTimestamp });
            // make sure the staking operation happens as soon as possible
            await stakeAtTimestamp(
                erc20DistributionInstance,
                firstStakerAddress,
                stakedAmount,
                startingTimestamp
            );
            await fastForwardTo({ timestamp: endingTimestamp.add(new BN(1)) });
            // staker staked for all of the campaign's duration, but we only claim half of the first reward
            const halfFirstRewardsAmount = firstRewardsAmount.div(new BN(2));
            await erc20DistributionInstance.claim(
                [halfFirstRewardsAmount, 0],
                firstStakerAddress,
                { from: firstStakerAddress }
            );
            expect(
                await firstRewardTokenInstance.balanceOf(firstStakerAddress)
            ).to.be.equalBn(halfFirstRewardsAmount);
            expect(
                await secondRewardTokenInstance.balanceOf(firstStakerAddress)
            ).to.be.equalBn(ZERO_BN);
            expect(
                await firstRewardTokenInstance.balanceOf(
                    erc20DistributionInstance.address
                )
            ).to.be.equalBn(halfFirstRewardsAmount);
            expect(
                await secondRewardTokenInstance.balanceOf(
                    erc20DistributionInstance.address
                )
            ).to.be.equalBn(secondRewardsAmount);
        });

        it("should succeed when claiming zero first reward and all of the second reward", async () => {
            const stakedAmount = await toWei(20, stakableTokenInstance);
            const firstRewardsAmount = await toWei(
                10,
                firstRewardTokenInstance
            );
            const secondRewardsAmount = await toWei(
                20,
                secondRewardTokenInstance
            );
            const {
                erc20DistributionInstance,
                startingTimestamp,
                endingTimestamp,
            } = await initializeDistribution({
                from: ownerAddress,
                erc20DistributionFactoryInstance,
                stakableToken: stakableTokenInstance,
                rewardTokens: [
                    firstRewardTokenInstance,
                    secondRewardTokenInstance,
                ],
                rewardAmounts: [firstRewardsAmount, secondRewardsAmount],
                duration: 10,
            });
            await initializeStaker({
                erc20DistributionInstance,
                stakableTokenInstance,
                stakerAddress: firstStakerAddress,
                stakableAmount: stakedAmount,
            });
            await fastForwardTo({ timestamp: startingTimestamp });
            // make sure the staking operation happens as soon as possible
            await stakeAtTimestamp(
                erc20DistributionInstance,
                firstStakerAddress,
                stakedAmount,
                startingTimestamp
            );
            await fastForwardTo({ timestamp: endingTimestamp.add(new BN(1)) });
            // staker staked for all of the campaign's duration, but we only claim half of the second reward
            const halfSecondRewardsAmount = secondRewardsAmount.div(new BN(2));
            await erc20DistributionInstance.claim(
                [0, halfSecondRewardsAmount],
                firstStakerAddress,
                { from: firstStakerAddress }
            );
            expect(
                await firstRewardTokenInstance.balanceOf(firstStakerAddress)
            ).to.be.equalBn(ZERO_BN);
            expect(
                await secondRewardTokenInstance.balanceOf(firstStakerAddress)
            ).to.be.equalBn(halfSecondRewardsAmount);
            expect(
                await firstRewardTokenInstance.balanceOf(
                    erc20DistributionInstance.address
                )
            ).to.be.equalBn(firstRewardsAmount);
            expect(
                await secondRewardTokenInstance.balanceOf(
                    erc20DistributionInstance.address
                )
            ).to.be.equalBn(halfSecondRewardsAmount);
        });

        it("should succeed in claiming two multiple rewards if two stakers stake exactly the same amount at different times", async () => {
            const stakedAmount = await toWei(10, stakableTokenInstance);
            const duration = new BN(10);
            const firstRewardAmount = await toWei(10, firstRewardTokenInstance);
            const secondRewardAmount = await toWei(
                50,
                secondRewardTokenInstance
            );
            const {
                erc20DistributionInstance,
                startingTimestamp,
                endingTimestamp,
            } = await initializeDistribution({
                from: ownerAddress,
                erc20DistributionFactoryInstance,
                stakableToken: stakableTokenInstance,
                rewardTokens: [
                    firstRewardTokenInstance,
                    secondRewardTokenInstance,
                ],
                rewardAmounts: [firstRewardAmount, secondRewardAmount],
                duration,
            });
            await initializeStaker({
                erc20DistributionInstance,
                stakableTokenInstance,
                stakerAddress: firstStakerAddress,
                stakableAmount: stakedAmount,
            });
            await initializeStaker({
                erc20DistributionInstance,
                stakableTokenInstance,
                stakerAddress: secondStakerAddress,
                stakableAmount: stakedAmount,
            });
            await fastForwardTo({
                timestamp: startingTimestamp,
            });
            // make sure the staking operation happens as soon as possible
            await stakeAtTimestamp(
                erc20DistributionInstance,
                firstStakerAddress,
                stakedAmount,
                startingTimestamp
            );
            const firstStakerStartingTimestamp = await getEvmTimestamp();
            expect(firstStakerStartingTimestamp).to.be.equalBn(
                startingTimestamp
            );
            // make half of the distribution time pass
            await fastForwardTo({
                timestamp: startingTimestamp.add(new BN(5)),
            });
            await stakeAtTimestamp(
                erc20DistributionInstance,
                secondStakerAddress,
                stakedAmount,
                startingTimestamp.add(new BN(5))
            );
            const secondStakerStartingTimestamp = await getEvmTimestamp();
            expect(secondStakerStartingTimestamp).to.be.equalBn(
                startingTimestamp.add(new BN(5))
            );
            await fastForwardTo({ timestamp: endingTimestamp });
            const onchainStartingTimestamp = await erc20DistributionInstance.startingTimestamp();
            const onchainEndingTimestamp = await erc20DistributionInstance.endingTimestamp();
            expect(onchainStartingTimestamp).to.be.equalBn(startingTimestamp);
            expect(onchainEndingTimestamp).to.be.equalBn(endingTimestamp);
            expect(
                onchainEndingTimestamp.sub(onchainStartingTimestamp)
            ).to.be.equalBn(duration);
            // first staker staked for 10 seconds
            expect(
                onchainEndingTimestamp.sub(firstStakerStartingTimestamp)
            ).to.be.equalBn(new BN(10));
            // second staker staked for 5 seconds
            expect(
                onchainEndingTimestamp.sub(secondStakerStartingTimestamp)
            ).to.be.equalBn(new BN(5));
            const firstRewardPerSecond = firstRewardAmount.div(
                onchainEndingTimestamp.sub(onchainStartingTimestamp)
            );
            const secondRewardPerSecond = secondRewardAmount.div(
                onchainEndingTimestamp.sub(onchainStartingTimestamp)
            );
            // the first staker had all of the rewards for 5 seconds and half of them for 5
            const expectedFirstFirstStakerReward = firstRewardPerSecond
                .mul(new BN(5))
                .add(firstRewardPerSecond.mul(new BN(5)).div(new BN(2)));
            const expectedSecondFirstStakerReward = secondRewardPerSecond
                .mul(new BN(5))
                .add(secondRewardPerSecond.mul(new BN(5)).div(new BN(2)));
            // the second staker had half of the rewards for 5 seconds
            const expectedFirstSecondStakerReward = firstRewardPerSecond
                .div(new BN(2))
                .mul(new BN(5));
            const expectedSecondSecondStakerReward = secondRewardPerSecond
                .div(new BN(2))
                .mul(new BN(5));
            // first staker claiming/balance checking
            await erc20DistributionInstance.claimAll(firstStakerAddress, {
                from: firstStakerAddress,
            });
            expect(
                await firstRewardTokenInstance.balanceOf(firstStakerAddress)
            ).to.be.equalBn(expectedFirstFirstStakerReward);
            expect(
                await secondRewardTokenInstance.balanceOf(firstStakerAddress)
            ).to.be.equalBn(expectedSecondFirstStakerReward);
            // second staker claiming/balance checking
            await erc20DistributionInstance.claimAll(secondStakerAddress, {
                from: secondStakerAddress,
            });
            expect(
                await firstRewardTokenInstance.balanceOf(secondStakerAddress)
            ).to.be.equalBn(expectedFirstSecondStakerReward);
            expect(
                await secondRewardTokenInstance.balanceOf(secondStakerAddress)
            ).to.be.equalBn(expectedSecondSecondStakerReward);
        });

        it("should succeed in claiming three rewards if three stakers stake exactly the same amount at different times", async () => {
            const stakedAmount = await toWei(10, stakableTokenInstance);
            const duration = new BN(12);
            const firstRewardAmount = await toWei(12, firstRewardTokenInstance);
            const secondRewardAmount = await toWei(
                30,
                secondRewardTokenInstance
            );
            const {
                erc20DistributionInstance,
                startingTimestamp,
                endingTimestamp,
            } = await initializeDistribution({
                from: ownerAddress,
                erc20DistributionFactoryInstance,
                stakableToken: stakableTokenInstance,
                rewardTokens: [
                    firstRewardTokenInstance,
                    secondRewardTokenInstance,
                ],
                rewardAmounts: [firstRewardAmount, secondRewardAmount],
                duration,
            });
            await initializeStaker({
                erc20DistributionInstance,
                stakableTokenInstance,
                stakerAddress: firstStakerAddress,
                stakableAmount: stakedAmount,
            });
            await initializeStaker({
                erc20DistributionInstance,
                stakableTokenInstance,
                stakerAddress: secondStakerAddress,
                stakableAmount: stakedAmount,
            });
            await initializeStaker({
                erc20DistributionInstance,
                stakableTokenInstance,
                stakerAddress: thirdStakerAddress,
                stakableAmount: stakedAmount,
            });
            await fastForwardTo({
                timestamp: startingTimestamp,
            });
            // first staker stakes
            await stakeAtTimestamp(
                erc20DistributionInstance,
                firstStakerAddress,
                stakedAmount,
                startingTimestamp
            );
            const firstStakerStartingTimestamp = await getEvmTimestamp();
            expect(firstStakerStartingTimestamp).to.be.equalBn(
                startingTimestamp
            );
            await fastForwardTo({
                timestamp: startingTimestamp.add(new BN(6)),
            });
            // second staker stakes
            await stakeAtTimestamp(
                erc20DistributionInstance,
                secondStakerAddress,
                stakedAmount,
                startingTimestamp.add(new BN(6))
            );
            const secondStakerStartingTimestamp = await getEvmTimestamp();
            expect(secondStakerStartingTimestamp).to.be.equalBn(
                startingTimestamp.add(new BN(6))
            );
            await fastForwardTo({
                timestamp: secondStakerStartingTimestamp.add(new BN(3)),
            });
            // third staker stakes
            await stakeAtTimestamp(
                erc20DistributionInstance,
                thirdStakerAddress,
                stakedAmount,
                secondStakerStartingTimestamp.add(new BN(3))
            );
            const thirdStakerStartingTimestamp = await getEvmTimestamp();
            expect(thirdStakerStartingTimestamp).to.be.equalBn(
                secondStakerStartingTimestamp.add(new BN(3))
            );
            // make sure the distribution has ended
            await fastForwardTo({
                timestamp: endingTimestamp.add(new BN(10)),
            });
            const onchainStartingTimestamp = await erc20DistributionInstance.startingTimestamp();
            const onchainEndingTimestamp = await erc20DistributionInstance.endingTimestamp();
            expect(onchainStartingTimestamp).to.be.equalBn(startingTimestamp);
            expect(onchainEndingTimestamp).to.be.equalBn(endingTimestamp);
            expect(
                onchainEndingTimestamp.sub(onchainStartingTimestamp)
            ).to.be.equalBn(duration);

            // first staker staked for 12 seconds
            expect(
                onchainEndingTimestamp.sub(firstStakerStartingTimestamp)
            ).to.be.equalBn(new BN(12));
            // second staker staked for 6 seconds
            expect(
                onchainEndingTimestamp.sub(secondStakerStartingTimestamp)
            ).to.be.equalBn(new BN(6));
            // third staker staked for 3 seconds
            expect(
                onchainEndingTimestamp.sub(thirdStakerStartingTimestamp)
            ).to.be.equalBn(new BN(3));

            const firstRewardPerSecond = firstRewardAmount.div(duration);
            const secondRewardPerSecond = secondRewardAmount.div(duration);
            // the first staker had all of the rewards for 6 seconds,
            // half of them for 3 seconds and a third for 3 seconds
            const expectedFirstFirstStakerReward = firstRewardPerSecond
                .mul(new BN(6))
                .add(firstRewardPerSecond.mul(new BN(3)).div(new BN(2)))
                .add(firstRewardPerSecond.mul(new BN(3)).div(new BN(3)));
            const expectedSecondFirstStakerReward = secondRewardPerSecond
                .mul(new BN(6))
                .add(secondRewardPerSecond.mul(new BN(3)).div(new BN(2)))
                .add(secondRewardPerSecond.mul(new BN(3)).div(new BN(3)));
            // the second staker had half of the rewards for 6 seconds
            // and a third for 3 seconds
            const expectedFirstSecondStakerReward = firstRewardPerSecond
                .mul(new BN(3))
                .div(new BN(2))
                .add(firstRewardPerSecond.mul(new BN(3)).div(new BN(3)));
            const expectedSecondSecondStakerReward = secondRewardPerSecond
                .mul(new BN(3))
                .div(new BN(2))
                .add(secondRewardPerSecond.mul(new BN(3)).div(new BN(3)));
            // the third staker had a third of the rewards for 3 seconds
            // (math says that they'd simply get a full second reward for 3 seconds,
            // but let's do the calculation anyway for added clarity)
            const expectedFirstThirdStakerReward = firstRewardPerSecond
                .mul(new BN(3))
                .div(new BN(3));
            const expectedSecondThirdStakerReward = secondRewardPerSecond
                .mul(new BN(3))
                .div(new BN(3));

            // first staker claiming/balance checking
            await erc20DistributionInstance.claimAll(firstStakerAddress, {
                from: firstStakerAddress,
            });
            expect(
                await firstRewardTokenInstance.balanceOf(firstStakerAddress)
            ).to.be.closeBn(expectedFirstFirstStakerReward, MAXIMUM_VARIANCE);
            expect(
                await secondRewardTokenInstance.balanceOf(firstStakerAddress)
            ).to.be.closeBn(expectedSecondFirstStakerReward, MAXIMUM_VARIANCE);

            // second staker claim and rewards balance check
            await erc20DistributionInstance.claimAll(secondStakerAddress, {
                from: secondStakerAddress,
            });
            expect(
                await firstRewardTokenInstance.balanceOf(secondStakerAddress)
            ).to.be.closeBn(expectedFirstSecondStakerReward, MAXIMUM_VARIANCE);
            expect(
                await secondRewardTokenInstance.balanceOf(secondStakerAddress)
            ).to.be.closeBn(expectedSecondSecondStakerReward, MAXIMUM_VARIANCE);

            // third staker claim and rewards balance check
            await erc20DistributionInstance.claimAll(thirdStakerAddress, {
                from: thirdStakerAddress,
            });
            expect(
                await firstRewardTokenInstance.balanceOf(thirdStakerAddress)
            ).to.be.closeBn(expectedFirstThirdStakerReward, MAXIMUM_VARIANCE);
            expect(
                await secondRewardTokenInstance.balanceOf(thirdStakerAddress)
            ).to.be.closeBn(expectedSecondThirdStakerReward, MAXIMUM_VARIANCE);
        });

        it("should succeed in claiming a reward if a staker stakes when the distribution has already started", async () => {
            const stakedAmount = await toWei(10, stakableTokenInstance);
            const duration = new BN(10);
            const firstRewardsAmount = await toWei(
                10,
                firstRewardTokenInstance
            );
            const secondRewardsAmount = await toWei(
                20,
                secondRewardTokenInstance
            );
            const {
                erc20DistributionInstance,
                startingTimestamp,
                endingTimestamp,
            } = await initializeDistribution({
                from: ownerAddress,
                erc20DistributionFactoryInstance,
                stakableToken: stakableTokenInstance,
                rewardTokens: [
                    firstRewardTokenInstance,
                    secondRewardTokenInstance,
                ],
                rewardAmounts: [firstRewardsAmount, secondRewardsAmount],
                duration,
            });
            await initializeStaker({
                erc20DistributionInstance,
                stakableTokenInstance,
                stakerAddress: firstStakerAddress,
                stakableAmount: stakedAmount,
            });
            // fast forward to half of the distribution duration
            await fastForwardTo({
                timestamp: startingTimestamp.add(new BN(5)),
            });
            await stakeAtTimestamp(
                erc20DistributionInstance,
                firstStakerAddress,
                stakedAmount,
                startingTimestamp.add(new BN(5))
            );
            const stakerStartingTimestamp = await getEvmTimestamp();
            expect(stakerStartingTimestamp).to.be.equalBn(
                startingTimestamp.add(new BN(5))
            );
            await fastForwardTo({ timestamp: endingTimestamp });
            const onchainStartingTimestamp = await erc20DistributionInstance.startingTimestamp();
            const onchainEndingTimestamp = await erc20DistributionInstance.endingTimestamp();
            expect(onchainStartingTimestamp).to.be.equalBn(startingTimestamp);
            expect(onchainEndingTimestamp).to.be.equalBn(endingTimestamp);
            // the staker staked for half of the duration
            expect(
                onchainEndingTimestamp.sub(stakerStartingTimestamp)
            ).to.be.equalBn(new BN(5));
            // claim and rewards balance check
            await erc20DistributionInstance.claimAll(firstStakerAddress, {
                from: firstStakerAddress,
            });
            expect(
                await firstRewardTokenInstance.balanceOf(firstStakerAddress)
            ).to.be.equalBn(await toWei("5", firstRewardTokenInstance));
            expect(
                await secondRewardTokenInstance.balanceOf(firstStakerAddress)
            ).to.be.equalBn(await toWei("10", secondRewardTokenInstance));
        });

        it("should fail in claiming 0 rewards if a staker stakes at the last second (literally)", async () => {
            const stakedAmount = await toWei(10, stakableTokenInstance);
            const duration = new BN(10);
            const firstRewardsAmount = await toWei(
                10,
                firstRewardTokenInstance
            );
            const secondRewardsAmount = await toWei(
                10,
                secondRewardTokenInstance
            );
            const {
                endingTimestamp,
                erc20DistributionInstance,
            } = await initializeDistribution({
                from: ownerAddress,
                erc20DistributionFactoryInstance,
                stakableToken: stakableTokenInstance,
                rewardTokens: [
                    firstRewardTokenInstance,
                    secondRewardTokenInstance,
                ],
                rewardAmounts: [firstRewardsAmount, secondRewardsAmount],
                duration,
            });
            await initializeStaker({
                erc20DistributionInstance,
                stakableTokenInstance,
                stakerAddress: firstStakerAddress,
                stakableAmount: stakedAmount,
            });
            await fastForwardTo({
                timestamp: endingTimestamp.sub(new BN(1)),
            });
            const stakerStartingTimestamp = endingTimestamp;
            await stakeAtTimestamp(
                erc20DistributionInstance,
                firstStakerAddress,
                stakedAmount,
                stakerStartingTimestamp
            );
            expect(stakerStartingTimestamp).to.be.equalBn(
                await getEvmTimestamp()
            );
            await fastForwardTo({ timestamp: endingTimestamp });
            const campaignEndingTimestamp = await erc20DistributionInstance.endingTimestamp();
            expect(
                campaignEndingTimestamp.sub(stakerStartingTimestamp)
            ).to.be.equalBn(ZERO_BN);
            try {
                await erc20DistributionInstance.claimAll(firstStakerAddress, {
                    from: firstStakerAddress,
                });
                throw new Error("should have failed");
            } catch (error) {
                expect(error.message).to.contain("SRD23");
            }
        });

        it("should succeed in claiming one rewards if a staker stakes at the last valid distribution second", async () => {
            const stakedAmount = await toWei(10, stakableTokenInstance);
            const duration = new BN(10);
            const firstRewardsAmount = await toWei(
                10,
                firstRewardTokenInstance
            );
            const secondRewardsAmount = await toWei(
                20,
                secondRewardTokenInstance
            );
            const {
                endingTimestamp,
                erc20DistributionInstance,
            } = await initializeDistribution({
                from: ownerAddress,
                erc20DistributionFactoryInstance,
                stakableToken: stakableTokenInstance,
                rewardTokens: [
                    firstRewardTokenInstance,
                    secondRewardTokenInstance,
                ],
                rewardAmounts: [firstRewardsAmount, secondRewardsAmount],
                duration,
            });
            await initializeStaker({
                erc20DistributionInstance,
                stakableTokenInstance,
                stakerAddress: firstStakerAddress,
                stakableAmount: stakedAmount,
            });
            const stakerStartingTimestamp = endingTimestamp.sub(new BN(1));
            await fastForwardTo({ timestamp: stakerStartingTimestamp });
            await stakeAtTimestamp(
                erc20DistributionInstance,
                firstStakerAddress,
                stakedAmount,
                stakerStartingTimestamp
            );
            expect(stakerStartingTimestamp).to.be.equalBn(
                await getEvmTimestamp()
            );
            await fastForwardTo({ timestamp: endingTimestamp });
            const campaignEndingTimestamp = await erc20DistributionInstance.endingTimestamp();
            expect(
                campaignEndingTimestamp.sub(stakerStartingTimestamp)
            ).to.be.equalBn(new BN(1));

            const firstRewardPerSecond = firstRewardsAmount.div(duration);
            const secondRewardPerSecond = secondRewardsAmount.div(duration);
            await erc20DistributionInstance.claimAll(firstStakerAddress, {
                from: firstStakerAddress,
            });
            expect(
                await firstRewardTokenInstance.balanceOf(firstStakerAddress)
            ).to.be.closeBn(firstRewardPerSecond, MAXIMUM_VARIANCE);
            expect(
                await secondRewardTokenInstance.balanceOf(firstStakerAddress)
            ).to.be.closeBn(secondRewardPerSecond, MAXIMUM_VARIANCE);
        });

        it("should succeed in claiming two rewards if two stakers stake exactly the same amount at different times, and then the first staker withdraws a portion of his stake", async () => {
            const stakedAmount = await toWei(10, stakableTokenInstance);
            const duration = new BN(10);
            const firstRewardsAmount = await toWei(
                10,
                firstRewardTokenInstance
            );
            const secondRewardsAmount = await toWei(
                40,
                secondRewardTokenInstance
            );
            const {
                startingTimestamp,
                endingTimestamp,
                erc20DistributionInstance,
            } = await initializeDistribution({
                from: ownerAddress,
                erc20DistributionFactoryInstance,
                stakableToken: stakableTokenInstance,
                rewardTokens: [
                    firstRewardTokenInstance,
                    secondRewardTokenInstance,
                ],
                rewardAmounts: [firstRewardsAmount, secondRewardsAmount],
                duration,
            });
            await initializeStaker({
                erc20DistributionInstance,
                stakableTokenInstance,
                stakerAddress: firstStakerAddress,
                stakableAmount: stakedAmount,
            });
            await initializeStaker({
                erc20DistributionInstance,
                stakableTokenInstance,
                stakerAddress: secondStakerAddress,
                stakableAmount: stakedAmount,
            });
            await fastForwardTo({
                timestamp: startingTimestamp,
            });
            await stakeAtTimestamp(
                erc20DistributionInstance,
                firstStakerAddress,
                stakedAmount,
                startingTimestamp
            );
            const firstStakerStartingTimestamp = await getEvmTimestamp();
            expect(firstStakerStartingTimestamp).to.be.equalBn(
                startingTimestamp
            );
            await fastForwardTo({
                timestamp: startingTimestamp.add(new BN(5)),
            });
            await stakeAtTimestamp(
                erc20DistributionInstance,
                secondStakerAddress,
                stakedAmount,
                startingTimestamp.add(new BN(5))
            );
            const secondStakerStartingTimestamp = await getEvmTimestamp();
            expect(secondStakerStartingTimestamp).to.be.equalBn(
                startingTimestamp.add(new BN(5))
            );
            // first staker withdraws at the eight second
            await fastForwardTo({
                timestamp: secondStakerStartingTimestamp.add(new BN(3)),
            });
            await withdrawAtTimestamp(
                erc20DistributionInstance,
                firstStakerAddress,
                stakedAmount.div(new BN(2)),
                secondStakerStartingTimestamp.add(new BN(3))
            );
            const firstStakerWithdrawTimestamp = await getEvmTimestamp();
            expect(firstStakerWithdrawTimestamp).to.be.equalBn(
                secondStakerStartingTimestamp.add(new BN(3))
            );
            await fastForwardTo({ timestamp: endingTimestamp });
            const onchainStartingTimestamp = await erc20DistributionInstance.startingTimestamp();
            const onchainEndingTimestamp = await erc20DistributionInstance.endingTimestamp();
            expect(onchainEndingTimestamp).to.be.equalBn(endingTimestamp);
            expect(onchainStartingTimestamp).to.be.equalBn(startingTimestamp);
            expect(
                onchainEndingTimestamp.sub(onchainStartingTimestamp)
            ).to.be.equalBn(duration);
            // first staker staked for 10 seconds
            expect(
                onchainEndingTimestamp.sub(firstStakerStartingTimestamp)
            ).to.be.equalBn(new BN(10));
            // first staker withdrew at second 8, 2 seconds before the end
            expect(
                onchainEndingTimestamp.sub(firstStakerWithdrawTimestamp)
            ).to.be.equalBn(new BN(2));
            // second staker staked for 5 seconds
            expect(
                onchainEndingTimestamp.sub(secondStakerStartingTimestamp)
            ).to.be.equalBn(new BN(5));
            const firstRewardPerSecond = firstRewardsAmount.div(duration);
            const secondRewardPerSecond = secondRewardsAmount.div(duration);
            // the first staker had all of the rewards for 5 seconds, half of them for 3, and a third for 2
            const expectedFirstFirstStakerReward = firstRewardPerSecond
                .mul(new BN(5))
                .add(firstRewardPerSecond.mul(new BN(3)).div(new BN(2)))
                .add(firstRewardPerSecond.mul(new BN(2)).div(new BN(3)));
            const expectedSecondFirstStakerReward = secondRewardPerSecond
                .mul(new BN(5))
                .add(secondRewardPerSecond.mul(new BN(3)).div(new BN(2)))
                .add(secondRewardPerSecond.mul(new BN(2)).div(new BN(3)));
            // the second staker had half of the rewards for 3 seconds and two thirds for 2
            const expectedFirstSecondStakerReward = firstRewardPerSecond
                .div(new BN(2))
                .mul(new BN(3))
                .add(
                    firstRewardPerSecond
                        .mul(new BN(2))
                        .mul(new BN(2))
                        .div(new BN(3))
                );
            const expectedSecondSecondStakerReward = secondRewardPerSecond
                .div(new BN(2))
                .mul(new BN(3))
                .add(
                    secondRewardPerSecond
                        .mul(new BN(2))
                        .mul(new BN(2))
                        .div(new BN(3))
                );
            // first staker claim and rewards balance check
            await erc20DistributionInstance.claimAll(firstStakerAddress, {
                from: firstStakerAddress,
            });
            expect(
                await firstRewardTokenInstance.balanceOf(firstStakerAddress)
            ).to.be.closeBn(expectedFirstFirstStakerReward, MAXIMUM_VARIANCE);
            expect(
                await secondRewardTokenInstance.balanceOf(firstStakerAddress)
            ).to.be.closeBn(expectedSecondFirstStakerReward, MAXIMUM_VARIANCE);
            // second staker claim and rewards balance check
            await erc20DistributionInstance.claimAll(secondStakerAddress, {
                from: secondStakerAddress,
            });
            expect(
                await firstRewardTokenInstance.balanceOf(secondStakerAddress)
            ).to.be.closeBn(expectedFirstSecondStakerReward, MAXIMUM_VARIANCE);
            expect(
                await secondRewardTokenInstance.balanceOf(secondStakerAddress)
            ).to.be.closeBn(expectedSecondSecondStakerReward, MAXIMUM_VARIANCE);
        });

        it("should succeed in claiming two rewards if two stakers both stake at the last valid distribution second", async () => {
            const stakedAmount = await toWei(10, stakableTokenInstance);
            const duration = new BN(10);
            const firstRewardsAmount = await toWei(
                10,
                firstRewardTokenInstance
            );
            const secondRewardsAmount = await toWei(
                20,
                secondRewardTokenInstance
            );
            const {
                endingTimestamp,
                erc20DistributionInstance,
            } = await initializeDistribution({
                from: ownerAddress,
                erc20DistributionFactoryInstance,
                stakableToken: stakableTokenInstance,
                rewardTokens: [
                    firstRewardTokenInstance,
                    secondRewardTokenInstance,
                ],
                rewardAmounts: [firstRewardsAmount, secondRewardsAmount],
                duration,
            });
            await initializeStaker({
                erc20DistributionInstance,
                stakableTokenInstance,
                stakerAddress: firstStakerAddress,
                stakableAmount: stakedAmount,
            });
            await initializeStaker({
                erc20DistributionInstance,
                stakableTokenInstance,
                stakerAddress: secondStakerAddress,
                stakableAmount: stakedAmount,
            });
            await stopMining();
            const stakingTimestamp = endingTimestamp.sub(new BN(1));
            await fastForwardTo({
                timestamp: stakingTimestamp,
            });
            await stakeAtTimestamp(
                erc20DistributionInstance,
                firstStakerAddress,
                stakedAmount,
                stakingTimestamp
            );
            await stakeAtTimestamp(
                erc20DistributionInstance,
                secondStakerAddress,
                stakedAmount,
                stakingTimestamp
            );
            expect(await getEvmTimestamp()).to.be.equalBn(stakingTimestamp);
            await startMining();
            await fastForwardTo({ timestamp: endingTimestamp });

            const onchainEndingTimestamp = await erc20DistributionInstance.endingTimestamp();
            const onchainStartingTimestamp = await erc20DistributionInstance.startingTimestamp();
            expect(
                onchainEndingTimestamp.sub(onchainStartingTimestamp)
            ).to.be.equalBn(duration);
            expect(onchainEndingTimestamp.sub(stakingTimestamp)).to.be.equalBn(
                new BN(1)
            );

            const firstRewardPerSecond = firstRewardsAmount.div(duration);
            const secondRewardPerSecond = secondRewardsAmount.div(duration);
            // the first staker had half of the rewards for 1 second
            const expectedFirstFirstStakerReward = firstRewardPerSecond.div(
                new BN(2)
            );
            const expectedSecondFirstStakerReward = secondRewardPerSecond.div(
                new BN(2)
            );
            // the second staker had half of the rewards for 1 second
            const expectedFirstSecondStakerReward = firstRewardPerSecond.div(
                new BN(2)
            );
            const expectedSecondSecondStakerReward = secondRewardPerSecond.div(
                new BN(2)
            );

            await erc20DistributionInstance.claimAll(firstStakerAddress, {
                from: firstStakerAddress,
            });
            expect(
                await firstRewardTokenInstance.balanceOf(firstStakerAddress)
            ).to.be.closeBn(expectedFirstFirstStakerReward, MAXIMUM_VARIANCE);
            expect(
                await secondRewardTokenInstance.balanceOf(firstStakerAddress)
            ).to.be.closeBn(expectedSecondFirstStakerReward, MAXIMUM_VARIANCE);

            await erc20DistributionInstance.claimAll(secondStakerAddress, {
                from: secondStakerAddress,
            });
            expect(
                await firstRewardTokenInstance.balanceOf(secondStakerAddress)
            ).to.be.closeBn(expectedFirstSecondStakerReward, MAXIMUM_VARIANCE);
            expect(
                await secondRewardTokenInstance.balanceOf(secondStakerAddress)
            ).to.be.closeBn(expectedSecondSecondStakerReward, MAXIMUM_VARIANCE);
        });

        it("should succeed in claiming a reward if a staker stakes at second n and then increases their stake", async () => {
            const stakedAmount = await toWei(10, stakableTokenInstance);
            const duration = new BN(10);
            const firstRewardsAmount = await toWei(
                10,
                firstRewardTokenInstance
            );
            const secondRewardsAmount = await toWei(
                100,
                secondRewardTokenInstance
            );
            const amountPerStake = stakedAmount.div(new BN(2));
            const {
                startingTimestamp,
                endingTimestamp,
                erc20DistributionInstance,
            } = await initializeDistribution({
                from: ownerAddress,
                erc20DistributionFactoryInstance,
                stakableToken: stakableTokenInstance,
                rewardTokens: [
                    firstRewardTokenInstance,
                    secondRewardTokenInstance,
                ],
                rewardAmounts: [firstRewardsAmount, secondRewardsAmount],
                duration,
            });
            await initializeStaker({
                erc20DistributionInstance,
                stakableTokenInstance,
                stakerAddress: firstStakerAddress,
                stakableAmount: stakedAmount,
            });
            await fastForwardTo({
                timestamp: startingTimestamp,
            });
            await stakeAtTimestamp(
                erc20DistributionInstance,
                firstStakerAddress,
                amountPerStake,
                startingTimestamp
            );
            const firstStakeStartingTimestamp = await getEvmTimestamp();
            expect(firstStakeStartingTimestamp).to.be.equalBn(
                startingTimestamp
            );
            await fastForwardTo({
                timestamp: startingTimestamp.add(new BN(5)),
            });
            await stakeAtTimestamp(
                erc20DistributionInstance,
                firstStakerAddress,
                amountPerStake,
                startingTimestamp.add(new BN(5))
            );
            const secondStakeStartingTimestamp = await getEvmTimestamp();
            await fastForwardTo({ timestamp: endingTimestamp });
            const onchainEndingTimestamp = await erc20DistributionInstance.endingTimestamp();
            const onchainStartingTimestamp = await erc20DistributionInstance.startingTimestamp();
            expect(
                onchainEndingTimestamp.sub(onchainStartingTimestamp)
            ).to.be.equalBn(duration);
            expect(
                onchainEndingTimestamp.sub(firstStakeStartingTimestamp)
            ).to.be.equalBn(new BN(10));
            expect(
                onchainEndingTimestamp.sub(secondStakeStartingTimestamp)
            ).to.be.equalBn(new BN(5));
            await erc20DistributionInstance.claimAll(firstStakerAddress, {
                from: firstStakerAddress,
            });
            expect(
                await firstRewardTokenInstance.balanceOf(firstStakerAddress)
            ).to.be.equalBn(firstRewardsAmount);
            expect(
                await secondRewardTokenInstance.balanceOf(firstStakerAddress)
            ).to.be.equalBn(secondRewardsAmount);
        });

        it("should succeed in claiming two rewards if two staker respectively stake and withdraw at the same second", async () => {
            const stakedAmount = await toWei(10, stakableTokenInstance);
            const duration = new BN(10);
            const firstRewardsAmount = await toWei(
                10,
                firstRewardTokenInstance
            );
            const secondRewardsAmount = await toWei(
                10,
                secondRewardTokenInstance
            );
            const {
                startingTimestamp,
                endingTimestamp,
                erc20DistributionInstance,
            } = await initializeDistribution({
                from: ownerAddress,
                erc20DistributionFactoryInstance,
                stakableToken: stakableTokenInstance,
                rewardTokens: [
                    firstRewardTokenInstance,
                    secondRewardTokenInstance,
                ],
                rewardAmounts: [firstRewardsAmount, secondRewardsAmount],
                duration,
            });
            await initializeStaker({
                erc20DistributionInstance,
                stakableTokenInstance,
                stakerAddress: firstStakerAddress,
                stakableAmount: stakedAmount,
            });
            await initializeStaker({
                erc20DistributionInstance,
                stakableTokenInstance,
                stakerAddress: secondStakerAddress,
                stakableAmount: stakedAmount,
            });
            await fastForwardTo({ timestamp: startingTimestamp });
            await stakeAtTimestamp(
                erc20DistributionInstance,
                firstStakerAddress,
                stakedAmount,
                startingTimestamp
            );
            const firstStakerStartingTimestamp = await getEvmTimestamp();
            expect(firstStakerStartingTimestamp).to.be.equalBn(
                startingTimestamp
            );
            await stopMining();
            const stakeAndWithdrawTimestamp = startingTimestamp.add(new BN(5));
            await fastForwardTo({
                timestamp: stakeAndWithdrawTimestamp,
            });
            await stakeAtTimestamp(
                erc20DistributionInstance,
                secondStakerAddress,
                stakedAmount,
                stakeAndWithdrawTimestamp
            );
            await withdrawAtTimestamp(
                erc20DistributionInstance,
                firstStakerAddress,
                stakedAmount,
                stakeAndWithdrawTimestamp
            );
            const secondStakerStartingTimestamp = await getEvmTimestamp();
            const firstStakerWithdrawTimestamp = await getEvmTimestamp();
            await startMining();
            expect(secondStakerStartingTimestamp).to.be.equalBn(
                stakeAndWithdrawTimestamp
            );
            expect(firstStakerWithdrawTimestamp).to.be.equalBn(
                stakeAndWithdrawTimestamp
            );
            await fastForwardTo({ timestamp: endingTimestamp });
            const onchainEndingTimestamp = await erc20DistributionInstance.endingTimestamp();
            const onchainStartingTimestamp = await erc20DistributionInstance.startingTimestamp();
            expect(
                onchainEndingTimestamp.sub(onchainStartingTimestamp)
            ).to.be.equalBn(duration);
            expect(
                firstStakerWithdrawTimestamp.sub(firstStakerStartingTimestamp)
            ).to.be.equalBn(new BN(5));
            expect(
                onchainEndingTimestamp.sub(secondStakerStartingTimestamp)
            ).to.be.equalBn(new BN(5));

            const firstRewardPerSecond = firstRewardsAmount.div(duration);
            const secondRewardPerSecond = secondRewardsAmount.div(duration);
            // both stakers had all of the rewards for 5 seconds
            const expectedFirstReward = firstRewardPerSecond.mul(new BN(5));
            const expectedSecondReward = secondRewardPerSecond.mul(new BN(5));

            // first staker claim and rewards balance check
            await erc20DistributionInstance.claimAll(firstStakerAddress, {
                from: firstStakerAddress,
            });
            expect(
                await firstRewardTokenInstance.balanceOf(firstStakerAddress)
            ).to.be.equalBn(expectedFirstReward);
            expect(
                await secondRewardTokenInstance.balanceOf(firstStakerAddress)
            ).to.be.equalBn(expectedSecondReward);

            // second staker claim and rewards balance check
            await erc20DistributionInstance.claimAll(secondStakerAddress, {
                from: secondStakerAddress,
            });
            expect(
                await firstRewardTokenInstance.balanceOf(secondStakerAddress)
            ).to.be.equalBn(expectedFirstReward);
            expect(
                await secondRewardTokenInstance.balanceOf(secondStakerAddress)
            ).to.be.equalBn(expectedSecondReward);
        });

        it("should fail when trying to claim passing an excessive-length amounts array", async () => {
            const duration = new BN(10);
            const {
                startingTimestamp,
                erc20DistributionInstance,
            } = await initializeDistribution({
                from: ownerAddress,
                erc20DistributionFactoryInstance,
                stakableToken: stakableTokenInstance,
                rewardTokens: [
                    firstRewardTokenInstance,
                    secondRewardTokenInstance,
                ],
                rewardAmounts: [
                    await toWei(10, firstRewardTokenInstance),
                    await toWei(10, secondRewardTokenInstance),
                ],
                duration,
            });
            await fastForwardTo({ timestamp: startingTimestamp });
            try {
                await erc20DistributionInstance.claim(
                    [new BN(1000), new BN(1000), new BN(1000)],
                    firstStakerAddress
                );
                throw new Error("should have failed");
            } catch (error) {
                expect(error.message).to.contain("SRD14");
            }
        });

        it("should fail when trying to claim passing a defective-length amounts array", async () => {
            const duration = new BN(10);
            const {
                startingTimestamp,
                erc20DistributionInstance,
            } = await initializeDistribution({
                from: ownerAddress,
                erc20DistributionFactoryInstance,
                stakableToken: stakableTokenInstance,
                rewardTokens: [
                    firstRewardTokenInstance,
                    secondRewardTokenInstance,
                ],
                rewardAmounts: [
                    await toWei(10, secondRewardTokenInstance),
                    await toWei(10, secondRewardTokenInstance),
                ],
                duration,
            });
            await fastForwardTo({ timestamp: startingTimestamp });
            try {
                await erc20DistributionInstance.claim(
                    [new BN(1000)],
                    firstStakerAddress
                );
                throw new Error("should have failed");
            } catch (error) {
                expect(error.message).to.contain("SRD14");
            }
        });

        it("should fail when trying to claim only a part of the reward, if the first passed in amount is bigger than allowed", async () => {
            const stakedAmount = await toWei(10, stakableTokenInstance);
            const duration = new BN(10);
            const firstRewardsAmount = await toWei(
                10,
                firstRewardTokenInstance
            );
            const secondRewardsAmount = await toWei(
                10,
                secondRewardTokenInstance
            );
            const {
                startingTimestamp,
                endingTimestamp,
                erc20DistributionInstance,
            } = await initializeDistribution({
                from: ownerAddress,
                erc20DistributionFactoryInstance,
                stakableToken: stakableTokenInstance,
                rewardTokens: [
                    firstRewardTokenInstance,
                    secondRewardTokenInstance,
                ],
                rewardAmounts: [firstRewardsAmount, secondRewardsAmount],
                duration,
            });
            await initializeStaker({
                erc20DistributionInstance,
                stakableTokenInstance,
                stakerAddress: firstStakerAddress,
                stakableAmount: stakedAmount,
            });
            await initializeStaker({
                erc20DistributionInstance,
                stakableTokenInstance,
                stakerAddress: secondStakerAddress,
                stakableAmount: stakedAmount,
            });
            await fastForwardTo({ timestamp: startingTimestamp });
            await stakeAtTimestamp(
                erc20DistributionInstance,
                firstStakerAddress,
                stakedAmount,
                startingTimestamp
            );
            await fastForwardTo({ timestamp: endingTimestamp });
            try {
                await erc20DistributionInstance.claim(
                    [
                        firstRewardsAmount.add(new BN(1000)),
                        secondRewardsAmount.sub(new BN(1000)),
                    ],
                    firstStakerAddress
                );
                throw new Error("should have failed");
            } catch (error) {
                expect(error.message).to.contain("SRD15");
            }
        });

        it("should fail when trying to claim only a part of the reward, if the second passed in amount is bigger than allowed", async () => {
            const stakedAmount = await toWei(10, stakableTokenInstance);
            const duration = new BN(10);
            const firstRewardsAmount = await toWei(
                10,
                firstRewardTokenInstance
            );
            const secondRewardsAmount = await toWei(
                10,
                secondRewardTokenInstance
            );
            const {
                startingTimestamp,
                endingTimestamp,
                erc20DistributionInstance,
            } = await initializeDistribution({
                from: ownerAddress,
                erc20DistributionFactoryInstance,
                stakableToken: stakableTokenInstance,
                rewardTokens: [
                    firstRewardTokenInstance,
                    secondRewardTokenInstance,
                ],
                rewardAmounts: [firstRewardsAmount, secondRewardsAmount],
                duration,
            });
            await initializeStaker({
                erc20DistributionInstance,
                stakableTokenInstance,
                stakerAddress: firstStakerAddress,
                stakableAmount: stakedAmount,
            });
            await initializeStaker({
                erc20DistributionInstance,
                stakableTokenInstance,
                stakerAddress: secondStakerAddress,
                stakableAmount: stakedAmount,
            });
            await fastForwardTo({ timestamp: startingTimestamp });
            await stakeAtTimestamp(
                erc20DistributionInstance,
                firstStakerAddress,
                stakedAmount,
                startingTimestamp
            );
            await fastForwardTo({ timestamp: endingTimestamp });
            try {
                await erc20DistributionInstance.claim(
                    [firstRewardsAmount, secondRewardsAmount.add(new BN(1000))],
                    firstStakerAddress
                );
                throw new Error("should have failed");
            } catch (error) {
                expect(error.message).to.contain("SRD15");
            }
        });

        it("should fail when trying to claim only a part of the reward, if the second passed in amount is bigger than allowed", async () => {
            const stakedAmount = await toWei(10, stakableTokenInstance);
            const duration = new BN(10);
            const firstRewardsAmount = await toWei(
                10,
                firstRewardTokenInstance
            );
            const secondRewardsAmount = await toWei(
                10,
                secondRewardTokenInstance
            );
            const {
                startingTimestamp,
                endingTimestamp,
                erc20DistributionInstance,
            } = await initializeDistribution({
                from: ownerAddress,
                erc20DistributionFactoryInstance,
                stakableToken: stakableTokenInstance,
                rewardTokens: [
                    firstRewardTokenInstance,
                    secondRewardTokenInstance,
                ],
                rewardAmounts: [firstRewardsAmount, secondRewardsAmount],
                duration,
            });
            await initializeStaker({
                erc20DistributionInstance,
                stakableTokenInstance,
                stakerAddress: firstStakerAddress,
                stakableAmount: stakedAmount,
            });
            await initializeStaker({
                erc20DistributionInstance,
                stakableTokenInstance,
                stakerAddress: secondStakerAddress,
                stakableAmount: stakedAmount,
            });
            await fastForwardTo({ timestamp: startingTimestamp });
            await stakeAtTimestamp(
                erc20DistributionInstance,
                firstStakerAddress,
                stakedAmount,
                startingTimestamp
            );
            await fastForwardTo({ timestamp: endingTimestamp });
            try {
                await erc20DistributionInstance.claim(
                    [firstRewardsAmount, secondRewardsAmount.add(new BN(1000))],
                    firstStakerAddress
                );
                throw new Error("should have failed");
            } catch (error) {
                expect(error.message).to.contain("SRD15");
            }
        });

        it("should fail when trying to claim only a part of the reward, if the second passed in amount is bigger than allowed", async () => {
            const stakedAmount = await toWei(10, stakableTokenInstance);
            const duration = new BN(10);
            const firstRewardsAmount = await toWei(
                10,
                firstRewardTokenInstance
            );
            const secondRewardsAmount = await toWei(
                10,
                secondRewardTokenInstance
            );
            const {
                startingTimestamp,
                endingTimestamp,
                erc20DistributionInstance,
            } = await initializeDistribution({
                from: ownerAddress,
                erc20DistributionFactoryInstance,
                stakableToken: stakableTokenInstance,
                rewardTokens: [
                    firstRewardTokenInstance,
                    secondRewardTokenInstance,
                ],
                rewardAmounts: [firstRewardsAmount, secondRewardsAmount],
                duration,
            });
            await initializeStaker({
                erc20DistributionInstance,
                stakableTokenInstance,
                stakerAddress: firstStakerAddress,
                stakableAmount: stakedAmount,
            });
            await initializeStaker({
                erc20DistributionInstance,
                stakableTokenInstance,
                stakerAddress: secondStakerAddress,
                stakableAmount: stakedAmount,
            });
            await fastForwardTo({ timestamp: startingTimestamp });
            await stakeAtTimestamp(
                erc20DistributionInstance,
                firstStakerAddress,
                stakedAmount,
                startingTimestamp
            );
            await fastForwardTo({ timestamp: endingTimestamp });
            try {
                await erc20DistributionInstance.claim(
                    [firstRewardsAmount, secondRewardsAmount.add(new BN(1000))],
                    firstStakerAddress
                );
                throw new Error("should have failed");
            } catch (error) {
                expect(error.message).to.contain("SRD15");
            }
        });

        it("should succeed in claiming specific amounts under the right conditions", async () => {
            const stakedAmount = await toWei(10, stakableTokenInstance);
            const duration = new BN(10);
            const firstRewardsAmount = await toWei(
                10,
                firstRewardTokenInstance
            );
            const secondRewardsAmount = await toWei(
                10,
                secondRewardTokenInstance
            );
            const {
                startingTimestamp,
                endingTimestamp,
                erc20DistributionInstance,
            } = await initializeDistribution({
                from: ownerAddress,
                erc20DistributionFactoryInstance,
                stakableToken: stakableTokenInstance,
                rewardTokens: [
                    firstRewardTokenInstance,
                    secondRewardTokenInstance,
                ],
                rewardAmounts: [firstRewardsAmount, secondRewardsAmount],
                duration,
            });
            await initializeStaker({
                erc20DistributionInstance,
                stakableTokenInstance,
                stakerAddress: firstStakerAddress,
                stakableAmount: stakedAmount,
            });
            await initializeStaker({
                erc20DistributionInstance,
                stakableTokenInstance,
                stakerAddress: secondStakerAddress,
                stakableAmount: stakedAmount,
            });
            await fastForwardTo({ timestamp: startingTimestamp });
            await stakeAtTimestamp(
                erc20DistributionInstance,
                firstStakerAddress,
                stakedAmount,
                startingTimestamp
            );
            await fastForwardTo({ timestamp: endingTimestamp });
            await erc20DistributionInstance.claim(
                [firstRewardsAmount, secondRewardsAmount],
                firstStakerAddress,
                { from: firstStakerAddress }
            );
            expect(
                await firstRewardTokenInstance.balanceOf(firstStakerAddress)
            ).to.be.equalBn(firstRewardsAmount);
            expect(
                await secondRewardTokenInstance.balanceOf(firstStakerAddress)
            ).to.be.equalBn(secondRewardsAmount);
        });

        it("should succeed in claiming specific amounts to a foreign address under the right conditions", async () => {
            const stakedAmount = await toWei(10, stakableTokenInstance);
            const duration = new BN(10);
            const firstRewardsAmount = await toWei(
                10,
                firstRewardTokenInstance
            );
            const secondRewardsAmount = await toWei(
                10,
                secondRewardTokenInstance
            );
            const {
                startingTimestamp,
                endingTimestamp,
                erc20DistributionInstance,
            } = await initializeDistribution({
                from: ownerAddress,
                erc20DistributionFactoryInstance,
                stakableToken: stakableTokenInstance,
                rewardTokens: [
                    firstRewardTokenInstance,
                    secondRewardTokenInstance,
                ],
                rewardAmounts: [firstRewardsAmount, secondRewardsAmount],
                duration,
            });
            await initializeStaker({
                erc20DistributionInstance,
                stakableTokenInstance,
                stakerAddress: firstStakerAddress,
                stakableAmount: stakedAmount,
            });
            await initializeStaker({
                erc20DistributionInstance,
                stakableTokenInstance,
                stakerAddress: secondStakerAddress,
                stakableAmount: stakedAmount,
            });
            await fastForwardTo({ timestamp: startingTimestamp });
            await stakeAtTimestamp(
                erc20DistributionInstance,
                firstStakerAddress,
                stakedAmount,
                startingTimestamp
            );
            await fastForwardTo({ timestamp: endingTimestamp });
            await erc20DistributionInstance.claim(
                [firstRewardsAmount, secondRewardsAmount],
                secondStakerAddress,
                { from: firstStakerAddress }
            );
            expect(
                await firstRewardTokenInstance.balanceOf(secondStakerAddress)
            ).to.be.equalBn(firstRewardsAmount);
            expect(
                await secondRewardTokenInstance.balanceOf(secondStakerAddress)
            ).to.be.equalBn(secondRewardsAmount);
        });
    }
);
```
#### test/erc20-distribution/multi-rewards-single-stakable/initialization.test.js
```javascript
const BN = require("bn.js");
const { expect } = require("chai");
const { ZERO_ADDRESS } = require("../../constants");
const { initializeDistribution } = require("../../utils");
const { toWei } = require("../../utils/conversion");
const { getEvmTimestamp } = require("../../utils/network");

const ERC20StakingRewardsDistribution = artifacts.require(
    "ERC20StakingRewardsDistribution"
);
const ERC20StakingRewardsDistributionFactory = artifacts.require(
    "ERC20StakingRewardsDistributionFactory"
);
const FirstRewardERC20 = artifacts.require("FirstRewardERC20");
const SecondRewardERC20 = artifacts.require("SecondRewardERC20");
const FirstStakableERC20 = artifacts.require("FirstStakableERC20");

contract(
    "ERC20StakingRewardsDistribution - Multi rewards, single stakable token - Initialization",
    () => {
        let erc20DistributionFactoryInstance,
            firstRewardsTokenInstance,
            secondRewardsTokenInstance,
            stakableTokenInstance,
            ownerAddress;

        beforeEach(async () => {
            const accounts = await web3.eth.getAccounts();
            ownerAddress = accounts[0];
            const erc20DistributionInstance = await ERC20StakingRewardsDistribution.new(
                { from: ownerAddress }
            );
            erc20DistributionFactoryInstance = await ERC20StakingRewardsDistributionFactory.new(
                erc20DistributionInstance.address,
                { from: ownerAddress }
            );
            firstRewardsTokenInstance = await FirstRewardERC20.new();
            secondRewardsTokenInstance = await SecondRewardERC20.new();
            stakableTokenInstance = await FirstStakableERC20.new();
        });

        it("should fail when reward tokens/amounts arrays have inconsistent lengths", async () => {
            try {
                const erc20DistributionInstance = await ERC20StakingRewardsDistribution.new(
                    { from: ownerAddress }
                );
                await erc20DistributionInstance.initialize(
                    [
                        firstRewardsTokenInstance.address,
                        secondRewardsTokenInstance.address,
                    ],
                    stakableTokenInstance.address,
                    [11],
                    100000000000,
                    1000000000000,
                    false,
                    0
                );
                throw new Error("should have failed");
            } catch (error) {
                expect(error.message).to.contain("SRD03");
            }
        });

        it("should fail when funding for the first reward token has not been sent to the contract before calling initialize", async () => {
            try {
                const erc20DistributionInstance = await ERC20StakingRewardsDistribution.new(
                    { from: ownerAddress }
                );
                await erc20DistributionInstance.initialize(
                    [
                        firstRewardsTokenInstance.address,
                        secondRewardsTokenInstance.address,
                    ],
                    stakableTokenInstance.address,
                    [11, 36],
                    100000000000,
                    1000000000000,
                    false,
                    0
                );
                throw new Error("should have failed");
            } catch (error) {
                expect(error.message).to.contain("SRD06");
            }
        });

        it("should fail when funding for the second reward token has not been sent to the contract before calling initialize", async () => {
            try {
                const erc20DistributionInstance = await ERC20StakingRewardsDistribution.new(
                    { from: ownerAddress }
                );
                await firstRewardsTokenInstance.mint(
                    erc20DistributionInstance.address,
                    11
                );
                await erc20DistributionInstance.initialize(
                    [
                        firstRewardsTokenInstance.address,
                        secondRewardsTokenInstance.address,
                    ],
                    stakableTokenInstance.address,
                    [11, 36],
                    100000000000,
                    1000000000000,
                    false,
                    0
                );
                throw new Error("should have failed");
            } catch (error) {
                expect(error.message).to.contain("SRD06");
            }
        });

        it("should fail when passing a 0-address second reward token", async () => {
            try {
                const erc20DistributionInstance = await ERC20StakingRewardsDistribution.new(
                    { from: ownerAddress }
                );
                await firstRewardsTokenInstance.mint(
                    erc20DistributionInstance.address,
                    11
                );
                await erc20DistributionInstance.initialize(
                    [firstRewardsTokenInstance.address, ZERO_ADDRESS],
                    stakableTokenInstance.address,
                    [11, 36],
                    100000000000,
                    1000000000000,
                    false,
                    0
                );
                throw new Error("should have failed");
            } catch (error) {
                expect(error.message).to.contain("SRD04");
            }
        });

        it("should fail when passing 0 as the first reward amount", async () => {
            try {
                await initializeDistribution({
                    from: ownerAddress,
                    erc20DistributionFactoryInstance,
                    stakableToken: stakableTokenInstance,
                    rewardTokens: [
                        firstRewardsTokenInstance,
                        secondRewardsTokenInstance,
                    ],
                    rewardAmounts: [0, 10],
                    duration: 10,
                });
                throw new Error("should have failed");
            } catch (error) {
                expect(error.message).to.contain("SRD05");
            }
        });

        it("should fail when passing 0 as the second reward amount", async () => {
            try {
                await initializeDistribution({
                    from: ownerAddress,
                    erc20DistributionFactoryInstance,
                    stakableToken: stakableTokenInstance,
                    rewardTokens: [
                        firstRewardsTokenInstance,
                        secondRewardsTokenInstance,
                    ],
                    rewardAmounts: [10, 0],
                    duration: 10,
                });
                throw new Error("should have failed");
            } catch (error) {
                expect(error.message).to.contain("SRD05");
            }
        });

        it("should succeed in the right conditions", async () => {
            const rewardAmounts = [
                new BN(await toWei(10, firstRewardsTokenInstance)),
                new BN(await toWei(100, secondRewardsTokenInstance)),
            ];
            const duration = new BN(10);
            const rewardTokens = [
                firstRewardsTokenInstance,
                secondRewardsTokenInstance,
            ];
            const {
                startingTimestamp,
                erc20DistributionInstance,
            } = await initializeDistribution({
                from: ownerAddress,
                erc20DistributionFactoryInstance,
                stakableToken: stakableTokenInstance,
                rewardTokens,
                rewardAmounts,
                duration,
            });

            expect(await erc20DistributionInstance.initialized()).to.be.true;
            const onchainRewardTokens = await erc20DistributionInstance.getRewardTokens();
            expect(onchainRewardTokens).to.have.length(2);
            expect(onchainRewardTokens[0]).to.be.equal(
                firstRewardsTokenInstance.address
            );
            expect(onchainRewardTokens[1]).to.be.equal(
                secondRewardsTokenInstance.address
            );
            const onchainStakableToken = await erc20DistributionInstance.stakableToken();
            expect(onchainStakableToken).to.be.equal(
                stakableTokenInstance.address
            );
            for (let i = 0; i < rewardTokens.length; i++) {
                const rewardAmount = rewardAmounts[i];
                const rewardToken = rewardTokens[i];
                expect(
                    await rewardToken.balanceOf(
                        erc20DistributionInstance.address
                    )
                ).to.be.equalBn(rewardAmount);
                expect(
                    await erc20DistributionInstance.rewardAmount(
                        rewardToken.address
                    )
                ).to.be.equalBn(rewardAmount);
            }
            const onchainStartingTimestamp = await erc20DistributionInstance.startingTimestamp();
            expect(onchainStartingTimestamp).to.be.equalBn(startingTimestamp);
            const onchainEndingTimestamp = await erc20DistributionInstance.endingTimestamp();
            expect(
                onchainEndingTimestamp.sub(onchainStartingTimestamp)
            ).to.be.equalBn(duration);
        });

        it("should fail when trying to initialize a second time", async () => {
            try {
                const {
                    erc20DistributionInstance,
                } = await initializeDistribution({
                    from: ownerAddress,
                    erc20DistributionFactoryInstance,
                    stakableToken: stakableTokenInstance,
                    rewardTokens: [
                        firstRewardsTokenInstance,
                        secondRewardsTokenInstance,
                    ],
                    rewardAmounts: [11, 14],
                    duration: 2,
                });
                const currentTimestamp = await getEvmTimestamp();
                await erc20DistributionInstance.initialize(
                    [
                        firstRewardsTokenInstance.address,
                        secondRewardsTokenInstance.address,
                    ],
                    stakableTokenInstance.address,
                    [17, 12],
                    currentTimestamp.add(new BN(10)),
                    currentTimestamp.add(new BN(20)),
                    false,
                    0
                );
                throw new Error("should have failed");
            } catch (error) {
                expect(error.message).to.contain("SRD18");
            }
        });
    }
);
```
#### test/erc20-distribution/multi-rewards-single-stakable/recovering.test.js
```javascript
const BN = require("bn.js");
const { expect } = require("chai");
const { ZERO_BN } = require("../../constants");
const {
    initializeDistribution,
    initializeStaker,
    stakeAtTimestamp,
    withdrawAtTimestamp,
} = require("../../utils");
const { toWei } = require("../../utils/conversion");
const {
    stopMining,
    startMining,
    fastForwardTo,
    getEvmTimestamp,
} = require("../../utils/network");

const ERC20StakingRewardsDistribution = artifacts.require(
    "ERC20StakingRewardsDistribution"
);
const ERC20StakingRewardsDistributionFactory = artifacts.require(
    "ERC20StakingRewardsDistributionFactory"
);
const FirstRewardERC20 = artifacts.require("FirstRewardERC20");
const SecondRewardERC20 = artifacts.require("SecondRewardERC20");
const FirstStakableERC20 = artifacts.require("FirstStakableERC20");

contract(
    "ERC20StakingRewardsDistribution - Multi rewards, single stakable token - Reward recovery",
    () => {
        let erc20DistributionFactoryInstance,
            firstRewardsTokenInstance,
            secondRewardsTokenInstance,
            stakableTokenInstance,
            ownerAddress,
            firstStakerAddress,
            secondStakerAddress;

        beforeEach(async () => {
            const accounts = await web3.eth.getAccounts();
            ownerAddress = accounts[0];
            const erc20DistributionInstance = await ERC20StakingRewardsDistribution.new(
                { from: ownerAddress }
            );
            erc20DistributionFactoryInstance = await ERC20StakingRewardsDistributionFactory.new(
                erc20DistributionInstance.address,
                { from: ownerAddress }
            );
            firstRewardsTokenInstance = await FirstRewardERC20.new();
            secondRewardsTokenInstance = await SecondRewardERC20.new();
            stakableTokenInstance = await FirstStakableERC20.new();
            firstStakerAddress = accounts[1];
            secondStakerAddress = accounts[2];
        });

        it("should recover all of the rewards when the distribution ended and no staker joined", async () => {
            const rewardTokens = [
                firstRewardsTokenInstance,
                secondRewardsTokenInstance,
            ];
            const rewardAmounts = [
                await toWei(100, firstRewardsTokenInstance),
                await toWei(10, secondRewardsTokenInstance),
            ];
            const {
                endingTimestamp,
                erc20DistributionInstance,
            } = await initializeDistribution({
                from: ownerAddress,
                erc20DistributionFactoryInstance,
                stakableToken: stakableTokenInstance,
                rewardTokens,
                rewardAmounts,
                duration: 10,
            });
            // at the start of the distribution, the owner deposited the rewards
            // into the staking contract, so their balance must be 0
            expect(
                await firstRewardsTokenInstance.balanceOf(ownerAddress)
            ).to.be.equalBn(ZERO_BN);
            expect(
                await secondRewardsTokenInstance.balanceOf(ownerAddress)
            ).to.be.equalBn(ZERO_BN);
            const onchainEndingTimestmp = await erc20DistributionInstance.endingTimestamp();
            expect(onchainEndingTimestmp).to.be.equalBn(endingTimestamp);
            await fastForwardTo({ timestamp: endingTimestamp });
            await erc20DistributionInstance.recoverUnassignedRewards();
            for (let i = 0; i < rewardAmounts.length; i++) {
                const rewardToken = rewardTokens[i];
                const rewardAmount = rewardAmounts[i];
                expect(await rewardToken.balanceOf(ownerAddress)).to.be.equalBn(
                    rewardAmount
                );
                expect(
                    await erc20DistributionInstance.recoverableUnassignedReward(
                        rewardToken.address
                    )
                ).to.be.equalBn(ZERO_BN);
            }
        });

        it("should always send funds to the contract's owner, even when called by another account", async () => {
            const rewardTokens = [
                firstRewardsTokenInstance,
                secondRewardsTokenInstance,
            ];
            const rewardAmounts = [
                await toWei(100, firstRewardsTokenInstance),
                await toWei(10, secondRewardsTokenInstance),
            ];
            const {
                endingTimestamp,
                erc20DistributionInstance,
            } = await initializeDistribution({
                from: ownerAddress,
                erc20DistributionFactoryInstance,
                stakableToken: stakableTokenInstance,
                rewardTokens,
                rewardAmounts,
                duration: 10,
            });
            // at the start of the distribution, the owner deposited the rewards
            // into the staking contract, so their balance must be 0
            expect(
                await firstRewardsTokenInstance.balanceOf(ownerAddress)
            ).to.be.equalBn(ZERO_BN);
            expect(
                await secondRewardsTokenInstance.balanceOf(ownerAddress)
            ).to.be.equalBn(ZERO_BN);
            await fastForwardTo({ timestamp: endingTimestamp });
            const onchainEndingTimestmp = await erc20DistributionInstance.endingTimestamp();
            expect(onchainEndingTimestmp).to.be.equalBn(endingTimestamp);
            await erc20DistributionInstance.recoverUnassignedRewards({
                from: firstStakerAddress,
            });
            for (let i = 0; i < rewardAmounts.length; i++) {
                const rewardToken = rewardTokens[i];
                const rewardAmount = rewardAmounts[i];
                expect(await rewardToken.balanceOf(ownerAddress)).to.be.equalBn(
                    rewardAmount
                );
                expect(
                    await rewardToken.balanceOf(firstStakerAddress)
                ).to.be.equalBn(ZERO_BN);
                expect(
                    await erc20DistributionInstance.recoverableUnassignedReward(
                        rewardToken.address
                    )
                ).to.be.equalBn(ZERO_BN);
            }
        });

        it("should recover half of the rewards when only one staker joined for half of the duration", async () => {
            const rewardTokens = [
                firstRewardsTokenInstance,
                secondRewardsTokenInstance,
            ];
            const rewardAmounts = [
                await toWei(10, firstRewardsTokenInstance),
                await toWei(100, secondRewardsTokenInstance),
            ];
            const {
                startingTimestamp,
                endingTimestamp,
                erc20DistributionInstance,
            } = await initializeDistribution({
                from: ownerAddress,
                erc20DistributionFactoryInstance,
                stakableToken: stakableTokenInstance,
                rewardTokens,
                rewardAmounts,
                duration: 10,
            });
            await initializeStaker({
                erc20DistributionInstance,
                stakableTokenInstance,
                stakerAddress: firstStakerAddress,
                stakableAmount: 1,
            });
            await fastForwardTo({ timestamp: startingTimestamp });
            expect(
                await firstRewardsTokenInstance.balanceOf(ownerAddress)
            ).to.be.equalBn(ZERO_BN);
            expect(
                await secondRewardsTokenInstance.balanceOf(ownerAddress)
            ).to.be.equalBn(ZERO_BN);
            // stake after 5 seconds until the end of the distribution
            const stakingStartingTimestamp = startingTimestamp.add(new BN(5));
            await fastForwardTo({ timestamp: stakingStartingTimestamp });
            await stakeAtTimestamp(
                erc20DistributionInstance,
                firstStakerAddress,
                [1],
                stakingStartingTimestamp
            );
            expect(await getEvmTimestamp()).to.be.equalBn(
                stakingStartingTimestamp
            );
            await fastForwardTo({ timestamp: endingTimestamp });
            const distributionEndingTimestamp = await erc20DistributionInstance.endingTimestamp();
            // staker staked for 5 seconds
            expect(
                distributionEndingTimestamp.sub(stakingStartingTimestamp)
            ).to.be.equalBn(new BN(5));
            // staker claims their reward
            const duration = endingTimestamp.sub(startingTimestamp);
            const firstRewardPerSecond = rewardAmounts[0].div(duration);
            const secondRewardPerSecond = rewardAmounts[1].div(duration);
            await erc20DistributionInstance.claimAll(firstStakerAddress, {
                from: firstStakerAddress,
            });
            expect(
                await firstRewardsTokenInstance.balanceOf(firstStakerAddress)
            ).to.be.equalBn(firstRewardPerSecond.mul(new BN(5)));
            expect(
                await secondRewardsTokenInstance.balanceOf(firstStakerAddress)
            ).to.be.equalBn(secondRewardPerSecond.mul(new BN(5)));
            await erc20DistributionInstance.recoverUnassignedRewards();
            expect(
                await firstRewardsTokenInstance.balanceOf(ownerAddress)
            ).to.be.equalBn(rewardAmounts[0].div(new BN(2)));
            expect(
                await secondRewardsTokenInstance.balanceOf(ownerAddress)
            ).to.be.equalBn(rewardAmounts[1].div(new BN(2)));
        });

        it("should recover half of the rewards when two stakers stake at the same time", async () => {
            const rewardTokens = [
                firstRewardsTokenInstance,
                secondRewardsTokenInstance,
            ];
            const rewardAmounts = [
                await toWei(10, firstRewardsTokenInstance),
                await toWei(100, secondRewardsTokenInstance),
            ];
            const {
                startingTimestamp,
                endingTimestamp,
                erc20DistributionInstance,
            } = await initializeDistribution({
                from: ownerAddress,
                erc20DistributionFactoryInstance,
                stakableToken: stakableTokenInstance,
                rewardTokens,
                rewardAmounts,
                duration: 20,
            });
            await initializeStaker({
                erc20DistributionInstance,
                stakableTokenInstance,
                stakerAddress: firstStakerAddress,
                stakableAmount: 1,
            });
            await initializeStaker({
                erc20DistributionInstance,
                stakableTokenInstance,
                stakerAddress: secondStakerAddress,
                stakableAmount: 1,
            });
            await fastForwardTo({ timestamp: startingTimestamp });
            expect(
                await firstRewardsTokenInstance.balanceOf(ownerAddress)
            ).to.be.equalBn(ZERO_BN);
            expect(
                await secondRewardsTokenInstance.balanceOf(ownerAddress)
            ).to.be.equalBn(ZERO_BN);
            // stake after 10 seconds until the end of the distribution
            const stakingTimestamp = startingTimestamp.add(new BN(10));
            await fastForwardTo({ timestamp: stakingTimestamp });
            await stopMining();
            await stakeAtTimestamp(
                erc20DistributionInstance,
                firstStakerAddress,
                [1],
                stakingTimestamp
            );
            await stakeAtTimestamp(
                erc20DistributionInstance,
                secondStakerAddress,
                [1],
                stakingTimestamp
            );
            expect(await getEvmTimestamp()).to.be.equalBn(stakingTimestamp);
            await startMining();
            await fastForwardTo({ timestamp: endingTimestamp });
            const onchainEndingTimestamp = await erc20DistributionInstance.endingTimestamp();
            // each staker staked for 10 seconds
            expect(onchainEndingTimestamp.sub(stakingTimestamp)).to.be.equalBn(
                new BN(10)
            );
            // stakers claim their reward
            const secondsDuration = await erc20DistributionInstance.secondsDuration();
            const firstRewardPerSecond = rewardAmounts[0].div(secondsDuration);
            const secondRewardPerSecond = rewardAmounts[1].div(secondsDuration);
            const expectedFirstReward = firstRewardPerSecond
                .div(new BN(2))
                .mul(new BN(10));
            const expectedSecondReward = secondRewardPerSecond
                .div(new BN(2))
                .mul(new BN(10));

            await erc20DistributionInstance.claimAll(firstStakerAddress, {
                from: firstStakerAddress,
            });
            expect(
                await firstRewardsTokenInstance.balanceOf(firstStakerAddress)
            ).to.be.equalBn(expectedFirstReward);
            expect(
                await secondRewardsTokenInstance.balanceOf(firstStakerAddress)
            ).to.be.equalBn(expectedSecondReward);

            await erc20DistributionInstance.claimAll(secondStakerAddress, {
                from: secondStakerAddress,
            });
            expect(
                await firstRewardsTokenInstance.balanceOf(secondStakerAddress)
            ).to.be.equalBn(expectedFirstReward);
            expect(
                await secondRewardsTokenInstance.balanceOf(secondStakerAddress)
            ).to.be.equalBn(expectedSecondReward);

            await erc20DistributionInstance.recoverUnassignedRewards();
            expect(
                await firstRewardsTokenInstance.balanceOf(ownerAddress)
            ).to.be.equalBn(rewardAmounts[0].div(new BN(2)));
            expect(
                await secondRewardsTokenInstance.balanceOf(ownerAddress)
            ).to.be.equalBn(rewardAmounts[1].div(new BN(2)));
        });

        it("should recover a third of the rewards when a staker stakes for two thirds of the distribution duration", async () => {
            const rewardTokens = [
                firstRewardsTokenInstance,
                secondRewardsTokenInstance,
            ];
            const rewardAmounts = [
                await toWei(10, firstRewardsTokenInstance),
                await toWei(100, secondRewardsTokenInstance),
            ];
            const {
                startingTimestamp,
                endingTimestamp,
                erc20DistributionInstance,
            } = await initializeDistribution({
                from: ownerAddress,
                erc20DistributionFactoryInstance,
                stakableToken: stakableTokenInstance,
                rewardTokens,
                rewardAmounts,
                duration: 12,
            });
            await initializeStaker({
                erc20DistributionInstance,
                stakableTokenInstance,
                stakerAddress: firstStakerAddress,
                stakableAmount: 1,
            });
            await fastForwardTo({ timestamp: startingTimestamp });
            expect(
                await firstRewardsTokenInstance.balanceOf(ownerAddress)
            ).to.be.equalBn(ZERO_BN);
            expect(
                await secondRewardsTokenInstance.balanceOf(ownerAddress)
            ).to.be.equalBn(ZERO_BN);
            // stake after 4 second until the end of the distribution
            const stakingTimestamp = startingTimestamp.add(new BN(4));
            await fastForwardTo({ timestamp: stakingTimestamp });
            await stakeAtTimestamp(
                erc20DistributionInstance,
                firstStakerAddress,
                [1],
                stakingTimestamp
            );
            expect(await getEvmTimestamp()).to.be.equalBn(stakingTimestamp);
            await fastForwardTo({ timestamp: endingTimestamp });
            const distributionEndingTimestamp = await erc20DistributionInstance.endingTimestamp();
            expect(
                distributionEndingTimestamp.sub(stakingTimestamp)
            ).to.be.equalBn(new BN(8));
            // staker claims their reward
            await erc20DistributionInstance.claimAll(firstStakerAddress, {
                from: firstStakerAddress,
            });
            // 6.6 should be claimable
            expect(
                await firstRewardsTokenInstance.balanceOf(firstStakerAddress)
            ).to.be.equalBn(new BN("6666666666666666666"));
            // 66.6 should be claimable
            expect(
                await secondRewardsTokenInstance.balanceOf(firstStakerAddress)
            ).to.be.equalBn(new BN("66666666666666666666"));
            await erc20DistributionInstance.recoverUnassignedRewards();
            expect(
                await firstRewardsTokenInstance.balanceOf(ownerAddress)
            ).to.be.equalBn(new BN("3333333333333333333"));
            expect(
                await secondRewardsTokenInstance.balanceOf(ownerAddress)
            ).to.be.equalBn(new BN("33333333333333333333"));
        });

        it("should recover two thirds of the rewards when a staker stakes for a third of the distribution duration, right in the middle", async () => {
            const rewardTokens = [
                firstRewardsTokenInstance,
                secondRewardsTokenInstance,
            ];
            const rewardAmounts = [
                await toWei(10, firstRewardsTokenInstance),
                await toWei(100, secondRewardsTokenInstance),
            ];
            const {
                startingTimestamp,
                endingTimestamp,
                erc20DistributionInstance,
            } = await initializeDistribution({
                from: ownerAddress,
                erc20DistributionFactoryInstance,
                stakableToken: stakableTokenInstance,
                rewardTokens,
                rewardAmounts,
                duration: 12,
            });
            await initializeStaker({
                erc20DistributionInstance,
                stakableTokenInstance,
                stakerAddress: firstStakerAddress,
                stakableAmount: 1,
            });
            await fastForwardTo({ timestamp: startingTimestamp });
            expect(
                await firstRewardsTokenInstance.balanceOf(ownerAddress)
            ).to.be.equalBn(ZERO_BN);
            expect(
                await secondRewardsTokenInstance.balanceOf(ownerAddress)
            ).to.be.equalBn(ZERO_BN);
            // stake after 4 seconds until the 8th second of the distribution (one third)
            const stakingTimestamp = startingTimestamp.add(new BN(4));
            await fastForwardTo({ timestamp: stakingTimestamp });
            await stakeAtTimestamp(
                erc20DistributionInstance,
                firstStakerAddress,
                [1],
                stakingTimestamp
            );
            expect(await getEvmTimestamp()).to.be.equalBn(stakingTimestamp);
            const withdrawTimestamp = startingTimestamp.add(new BN(8));
            await fastForwardTo({ timestamp: withdrawTimestamp });
            // withdraw after 4 seconds, occupying 4 seconds in total
            await withdrawAtTimestamp(
                erc20DistributionInstance,
                firstStakerAddress,
                [1],
                withdrawTimestamp
            );
            expect(await getEvmTimestamp()).to.be.equalBn(withdrawTimestamp);
            await fastForwardTo({ timestamp: endingTimestamp });

            expect(withdrawTimestamp.sub(stakingTimestamp)).to.be.equalBn(
                new BN(4)
            );
            // a third of the original reward
            const expectedFirstReward = rewardAmounts[0].div(new BN(3));
            const expectedSecondReward = rewardAmounts[1].div(new BN(3));
            // staker claims their reward
            await erc20DistributionInstance.claimAll(firstStakerAddress, {
                from: firstStakerAddress,
            });
            expect(
                await firstRewardsTokenInstance.balanceOf(firstStakerAddress)
            ).to.be.equalBn(expectedFirstReward);
            expect(
                await secondRewardsTokenInstance.balanceOf(firstStakerAddress)
            ).to.be.equalBn(expectedSecondReward);
            await erc20DistributionInstance.recoverUnassignedRewards();
            // expect two third of the reward to be recovered
            expect(
                await firstRewardsTokenInstance.balanceOf(ownerAddress)
            ).to.be.equalBn(new BN("6666666666666666666"));
            expect(
                await secondRewardsTokenInstance.balanceOf(ownerAddress)
            ).to.be.equalBn(new BN("66666666666666666666"));
        });

        it("should recover two thirds of the rewards when a staker stakes for a third of the distribution duration, in the end period", async () => {
            const rewardTokens = [
                firstRewardsTokenInstance,
                secondRewardsTokenInstance,
            ];
            const rewardAmounts = [
                await toWei(10, firstRewardsTokenInstance),
                await toWei(100, secondRewardsTokenInstance),
            ];
            const {
                startingTimestamp,
                endingTimestamp,
                erc20DistributionInstance,
            } = await initializeDistribution({
                from: ownerAddress,
                erc20DistributionFactoryInstance,
                stakableToken: stakableTokenInstance,
                rewardTokens,
                rewardAmounts,
                duration: 12,
            });
            await initializeStaker({
                erc20DistributionInstance,
                stakableTokenInstance,
                stakerAddress: firstStakerAddress,
                stakableAmount: 1,
            });
            await fastForwardTo({ timestamp: startingTimestamp });
            expect(
                await firstRewardsTokenInstance.balanceOf(ownerAddress)
            ).to.be.equalBn(ZERO_BN);
            expect(
                await secondRewardsTokenInstance.balanceOf(ownerAddress)
            ).to.be.equalBn(ZERO_BN);
            // stake after 8 second until the end of the distribution
            const stakingTimestamp = startingTimestamp.add(new BN(8));
            await fastForwardTo({ timestamp: stakingTimestamp });
            await stakeAtTimestamp(
                erc20DistributionInstance,
                firstStakerAddress,
                [1],
                stakingTimestamp
            );
            expect(await getEvmTimestamp()).to.be.equalBn(stakingTimestamp);
            await fastForwardTo({ timestamp: endingTimestamp });
            const distributionEndingTimestamp = await erc20DistributionInstance.endingTimestamp();
            expect(
                distributionEndingTimestamp.sub(stakingTimestamp)
            ).to.be.equalBn(new BN(4));
            // staker claims their reward
            await erc20DistributionInstance.claimAll(firstStakerAddress, {
                from: firstStakerAddress,
            });
            // should have claimed 3.3
            expect(
                await firstRewardsTokenInstance.balanceOf(firstStakerAddress)
            ).to.be.equalBn(new BN("3333333333333333333"));
            // should have claimed 33.3
            expect(
                await secondRewardsTokenInstance.balanceOf(firstStakerAddress)
            ).to.be.equalBn(new BN("33333333333333333333"));
            await erc20DistributionInstance.recoverUnassignedRewards();
            // should have recovered 6.6
            expect(
                await firstRewardsTokenInstance.balanceOf(ownerAddress)
            ).to.be.equalBn(new BN("6666666666666666666"));
            // should have recovered 66.6
            expect(
                await secondRewardsTokenInstance.balanceOf(ownerAddress)
            ).to.be.equalBn(new BN("66666666666666666666"));
        });
    }
);
```








#### test/erc20-distribution/single-tokens/cancelations.test.js
```javascript
const BN = require("bn.js");
const { expect } = require("chai");
const { ZERO_BN } = require("../../constants");
const { initializeDistribution } = require("../../utils");
const { toWei } = require("../../utils/conversion");
const { fastForwardTo, getEvmTimestamp } = require("../../utils/network");

const ERC20StakingRewardsDistribution = artifacts.require(
    "ERC20StakingRewardsDistribution"
);
const ERC20StakingRewardsDistributionFactory = artifacts.require(
    "ERC20StakingRewardsDistributionFactory"
);
const FirstRewardERC20 = artifacts.require("FirstRewardERC20");
const FirstStakableERC20 = artifacts.require("FirstStakableERC20");

contract(
    "ERC20StakingRewardsDistribution - Single reward/stakable token - Cancelation",
    () => {
        let erc20DistributionFactoryInstance,
            rewardsTokenInstance,
            stakableTokenInstance,
            ownerAddress,
            stakerAddress;

        beforeEach(async () => {
            const accounts = await web3.eth.getAccounts();
            ownerAddress = accounts[0];
            const erc20DistributionInstance = await ERC20StakingRewardsDistribution.new(
                { from: ownerAddress }
            );
            erc20DistributionFactoryInstance = await ERC20StakingRewardsDistributionFactory.new(
                erc20DistributionInstance.address,
                { from: ownerAddress }
            );
            rewardsTokenInstance = await FirstRewardERC20.new();
            stakableTokenInstance = await FirstStakableERC20.new();
            stakerAddress = accounts[1];
        });

        it("should fail when initialization has not been done", async () => {
            try {
                // initializing now sets the owner
                const {
                    erc20DistributionInstance,
                } = await initializeDistribution({
                    from: ownerAddress,
                    erc20DistributionFactoryInstance,
                    stakableToken: stakableTokenInstance,
                    rewardTokens: [rewardsTokenInstance],
                    rewardAmounts: [2],
                    duration: 2,
                });
                // canceling deinitializes the distribution
                await erc20DistributionInstance.cancel({ from: ownerAddress });
                await erc20DistributionInstance.cancel({ from: ownerAddress });
                throw new Error("should have failed");
            } catch (error) {
                expect(error.message).to.contain("SRD19");
            }
        });

        it("should fail when not called by the owner", async () => {
            try {
                const {
                    erc20DistributionInstance,
                } = await initializeDistribution({
                    from: ownerAddress,
                    erc20DistributionFactoryInstance,
                    stakableToken: stakableTokenInstance,
                    rewardTokens: [rewardsTokenInstance],
                    rewardAmounts: [2],
                    duration: 2,
                });
                await erc20DistributionInstance.cancel({ from: stakerAddress });
                throw new Error("should have failed");
            } catch (error) {
                expect(error.message).to.contain("SRD17");
            }
        });

        it("should fail when the program has already started", async () => {
            try {
                const {
                    startingTimestamp,
                    erc20DistributionInstance,
                } = await initializeDistribution({
                    from: ownerAddress,
                    erc20DistributionFactoryInstance,
                    stakableToken: stakableTokenInstance,
                    rewardTokens: [rewardsTokenInstance],
                    rewardAmounts: [5],
                    duration: 2,
                });
                await fastForwardTo({ timestamp: startingTimestamp });
                await erc20DistributionInstance.cancel({ from: ownerAddress });
                throw new Error("should have failed");
            } catch (error) {
                expect(error.message).to.contain("SRD08");
            }
        });

        it("should succeed in the right conditions", async () => {
            const rewardsAmount = await toWei(10, rewardsTokenInstance);
            const rewardTokens = [rewardsTokenInstance];
            const { erc20DistributionInstance } = await initializeDistribution({
                from: ownerAddress,
                erc20DistributionFactoryInstance,
                stakableToken: stakableTokenInstance,
                rewardTokens,
                rewardAmounts: [rewardsAmount],
                duration: 2,
                // future timestamp
                startingTimestamp: (await getEvmTimestamp()).add(new BN(60)),
            });
            await erc20DistributionInstance.cancel({ from: ownerAddress });
            for (let i = 0; i < rewardTokens.length; i++) {
                const rewardToken = rewardTokens[i];
                expect(
                    await rewardToken.balanceOf(
                        erc20DistributionInstance.address
                    )
                ).to.be.equalBn(ZERO_BN);
                expect(await rewardToken.balanceOf(ownerAddress)).to.be.equalBn(
                    rewardsAmount
                );
                expect(
                    await erc20DistributionInstance.rewardAmount(
                        rewardToken.address
                    )
                ).to.be.equalBn(rewardsAmount);
            }
            expect(await erc20DistributionInstance.initialized()).to.be.true;
            expect(await erc20DistributionInstance.canceled()).to.be.true;
        });

        it("shouldn't allow for a second initialization on success", async () => {
            const rewardsAmount = await toWei(10, rewardsTokenInstance);
            const { erc20DistributionInstance } = await initializeDistribution({
                from: ownerAddress,
                erc20DistributionFactoryInstance,
                stakableToken: stakableTokenInstance,
                rewardTokens: [rewardsTokenInstance],
                rewardAmounts: [rewardsAmount],
                duration: 2,
                // far-future timestamp
                startingTimestamp: (await getEvmTimestamp()).add(new BN(60)),
            });
            await erc20DistributionInstance.cancel({ from: ownerAddress });
            try {
                await erc20DistributionInstance.initialize(
                    [rewardsTokenInstance.address],
                    stakableTokenInstance.address,
                    [rewardsAmount],
                    1000000000000,
                    10000000000000,
                    false,
                    0
                );
                throw new Error("should have failed");
            } catch (error) {
                expect(error.message).to.contain("SRD18");
            }
        });

        it("shouldn't allow for a second cancelation on success", async () => {
            const rewardsAmount = await toWei(10, rewardsTokenInstance);
            const { erc20DistributionInstance } = await initializeDistribution({
                from: ownerAddress,
                erc20DistributionFactoryInstance,
                stakableToken: stakableTokenInstance,
                rewardTokens: [rewardsTokenInstance],
                rewardAmounts: [rewardsAmount],
                duration: 2,
                // far-future timestamp
                startingTimestamp: (await getEvmTimestamp()).add(new BN(60)),
            });
            await erc20DistributionInstance.cancel({ from: ownerAddress });
            try {
                await erc20DistributionInstance.cancel({ from: ownerAddress });
                throw new Error("should have failed");
            } catch (error) {
                expect(error.message).to.contain("SRD19");
            }
        });
    }
);

```
#### test/erc20-distribution/single-tokens/claimable-rewards.test.js
```javascript
const BN = require("bn.js");
const { expect } = require("chai");
const { initializeDistribution } = require("../../utils");

const ERC20StakingRewardsDistribution = artifacts.require(
    "ERC20StakingRewardsDistribution"
);
const ERC20StakingRewardsDistributionFactory = artifacts.require(
    "ERC20StakingRewardsDistributionFactory"
);
const FirstRewardERC20 = artifacts.require("FirstRewardERC20");
const SecondRewardERC20 = artifacts.require("SecondRewardERC20");
const FirstStakableERC20 = artifacts.require("FirstStakableERC20");

contract(
    "ERC20StakingRewardsDistribution - Single reward/stakable token - Get claimable rewards",
    () => {
        let erc20DistributionFactoryInstance,
            firstRewardTokenInstance,
            secondRewardTokenInstance,
            stakableTokenInstance,
            ownerAddress,
            firstStakerAddress;

        beforeEach(async () => {
            const accounts = await web3.eth.getAccounts();
            ownerAddress = accounts[0];
            const erc20DistributionInstance = await ERC20StakingRewardsDistribution.new(
                { from: ownerAddress }
            );
            erc20DistributionFactoryInstance = await ERC20StakingRewardsDistributionFactory.new(
                erc20DistributionInstance.address,
                { from: ownerAddress }
            );
            firstStakerAddress = accounts[1];
            firstRewardTokenInstance = await FirstRewardERC20.new();
            secondRewardTokenInstance = await SecondRewardERC20.new();
            stakableTokenInstance = await FirstStakableERC20.new();
            firstStakerAddress = accounts[1];
        });

        it("should give an empty array back when the distribution has not been initialized yet", async () => {
            const erc20DistributionInstance = await ERC20StakingRewardsDistribution.new(
                { from: ownerAddress }
            );
            const claimableRewards = await erc20DistributionInstance.claimableRewards(
                firstStakerAddress
            );
            expect(claimableRewards).to.have.length(0);
        });

        it("should give an array back with length 1 when the distribution has been initialized with 1 reward token but not yet started", async () => {
            const { erc20DistributionInstance } = await initializeDistribution({
                from: ownerAddress,
                erc20DistributionFactoryInstance,
                stakableToken: stakableTokenInstance,
                rewardTokens: [firstRewardTokenInstance],
                rewardAmounts: ["10"],
                duration: 10,
            });
            const claimableRewards = await erc20DistributionInstance.claimableRewards(
                firstStakerAddress
            );
            expect(claimableRewards).to.have.length(1);
            expect(claimableRewards[0]).to.be.equalBn(new BN(0));
        });

        it("should give an array back with length 2 when the distribution has been initialized with 2 reward token but not yet started", async () => {
            const { erc20DistributionInstance } = await initializeDistribution({
                from: ownerAddress,
                erc20DistributionFactoryInstance,
                stakableToken: stakableTokenInstance,
                rewardTokens: [
                    firstRewardTokenInstance,
                    secondRewardTokenInstance,
                ],
                rewardAmounts: ["10", "1"],
                duration: 10,
            });
            const claimableRewards = await erc20DistributionInstance.claimableRewards(
                firstStakerAddress
            );
            expect(claimableRewards).to.have.length(2);
            expect(claimableRewards[0]).to.be.equalBn(new BN(0));
            expect(claimableRewards[1]).to.be.equalBn(new BN(0));
        });
    }
);
```
#### test/erc20-distribution/single-tokens/claiming.test.js
```javascript
const BN = require("bn.js");
const { expect } = require("chai");
const { MAXIMUM_VARIANCE, ZERO_BN } = require("../../constants");
const {
    initializeDistribution,
    initializeStaker,
    stakeAtTimestamp,
    withdrawAtTimestamp,
    claimAllAtTimestamp,
} = require("../../utils");
const { toWei } = require("../../utils/conversion");
const {
    stopMining,
    startMining,
    fastForwardTo,
    getEvmTimestamp,
} = require("../../utils/network");

const ERC20StakingRewardsDistribution = artifacts.require(
    "ERC20StakingRewardsDistribution"
);
const ERC20StakingRewardsDistributionFactory = artifacts.require(
    "ERC20StakingRewardsDistributionFactory"
);
const FirstRewardERC20 = artifacts.require("FirstRewardERC20");
const FirstStakableERC20 = artifacts.require("FirstStakableERC20");

contract(
    "ERC20StakingRewardsDistribution - Single reward/stakable token - Claiming",
    () => {
        let erc20DistributionFactoryInstance,
            rewardsTokenInstance,
            stakableTokenInstance,
            ownerAddress,
            firstStakerAddress,
            secondStakerAddress,
            thirdStakerAddress;

        beforeEach(async () => {
            const accounts = await web3.eth.getAccounts();
            ownerAddress = accounts[0];
            const erc20DistributionInstance = await ERC20StakingRewardsDistribution.new(
                { from: ownerAddress }
            );
            erc20DistributionFactoryInstance = await ERC20StakingRewardsDistributionFactory.new(
                erc20DistributionInstance.address,
                { from: ownerAddress }
            );
            rewardsTokenInstance = await FirstRewardERC20.new();
            stakableTokenInstance = await FirstStakableERC20.new();
            firstStakerAddress = accounts[1];
            secondStakerAddress = accounts[2];
            thirdStakerAddress = accounts[3];
        });

        it("should succeed in claiming the full reward if only one staker stakes right from the first second", async () => {
            const stakedAmount = await toWei(20, stakableTokenInstance);
            const rewardsAmount = await toWei(10, rewardsTokenInstance);
            const {
                startingTimestamp,
                endingTimestamp,
                erc20DistributionInstance,
            } = await initializeDistribution({
                from: ownerAddress,
                erc20DistributionFactoryInstance,
                stakableToken: stakableTokenInstance,
                rewardTokens: [rewardsTokenInstance],
                rewardAmounts: [rewardsAmount],
                duration: 10,
            });
            await initializeStaker({
                erc20DistributionInstance,
                stakableTokenInstance,
                stakerAddress: firstStakerAddress,
                stakableAmount: stakedAmount,
            });
            await fastForwardTo({
                timestamp: startingTimestamp,
                mineBlockAfter: false,
            });
            await stakeAtTimestamp(
                erc20DistributionInstance,
                firstStakerAddress,
                stakedAmount,
                startingTimestamp
            );
            const stakerStartingTimestamp = await getEvmTimestamp();
            expect(stakerStartingTimestamp).to.be.equalBn(startingTimestamp);
            // make sure the distribution has ended
            await fastForwardTo({ timestamp: endingTimestamp.add(new BN(1)) });
            await erc20DistributionInstance.claimAll(firstStakerAddress, {
                from: firstStakerAddress,
            });
            const onchainStartingTimestamp = await erc20DistributionInstance.startingTimestamp();
            const onchainEndingTimestamp = await erc20DistributionInstance.endingTimestamp();
            expect(onchainStartingTimestamp).to.be.equalBn(startingTimestamp);
            expect(onchainEndingTimestamp).to.be.equalBn(endingTimestamp);
            const stakingDuration = onchainEndingTimestamp.sub(
                onchainStartingTimestamp
            );
            expect(stakingDuration).to.be.equalBn(new BN(10));
            expect(
                await rewardsTokenInstance.balanceOf(firstStakerAddress)
            ).to.equalBn(rewardsAmount);
        });

        it("should fail when claiming zero rewards (claimAll)", async () => {
            const stakedAmount = await toWei(20, stakableTokenInstance);
            const rewardsAmount = await toWei(10, rewardsTokenInstance);
            const {
                startingTimestamp,
                erc20DistributionInstance,
            } = await initializeDistribution({
                from: ownerAddress,
                erc20DistributionFactoryInstance,
                stakableToken: stakableTokenInstance,
                rewardTokens: [rewardsTokenInstance],
                rewardAmounts: [rewardsAmount],
                duration: 10,
            });
            await initializeStaker({
                erc20DistributionInstance,
                stakableTokenInstance,
                stakerAddress: firstStakerAddress,
                stakableAmount: stakedAmount,
            });
            await fastForwardTo({ timestamp: startingTimestamp });
            try {
                await erc20DistributionInstance.claimAll(firstStakerAddress, {
                    from: firstStakerAddress,
                });
                throw new Error("should have failed");
            } catch (error) {
                expect(error.message).to.contain("SRD23");
            }
        });

        it("should fail when claiming zero rewards (claim)", async () => {
            const stakedAmount = await toWei(20, stakableTokenInstance);
            const rewardsAmount = await toWei(10, rewardsTokenInstance);
            const {
                startingTimestamp,
                erc20DistributionInstance,
            } = await initializeDistribution({
                from: ownerAddress,
                erc20DistributionFactoryInstance,
                stakableToken: stakableTokenInstance,
                rewardTokens: [rewardsTokenInstance],
                rewardAmounts: [rewardsAmount],
                duration: 10,
            });
            await initializeStaker({
                erc20DistributionInstance,
                stakableTokenInstance,
                stakerAddress: firstStakerAddress,
                stakableAmount: stakedAmount,
            });
            await fastForwardTo({ timestamp: startingTimestamp });
            try {
                await erc20DistributionInstance.claim([0], firstStakerAddress, {
                    from: firstStakerAddress,
                });
                throw new Error("should have failed");
            } catch (error) {
                expect(error.message).to.contain("SRD24");
            }
        });

        it("should succeed in claiming two rewards if two stakers stake exactly the same amount at different times", async () => {
            const stakedAmount = await toWei(10, stakableTokenInstance);
            const duration = new BN(10);
            const rewardsAmount = await toWei(10, rewardsTokenInstance);
            const {
                startingTimestamp,
                endingTimestamp,
                erc20DistributionInstance,
            } = await initializeDistribution({
                from: ownerAddress,
                erc20DistributionFactoryInstance,
                stakableToken: stakableTokenInstance,
                rewardTokens: [rewardsTokenInstance],
                rewardAmounts: [rewardsAmount],
                duration,
            });
            await initializeStaker({
                erc20DistributionInstance,
                stakableTokenInstance,
                stakerAddress: firstStakerAddress,
                stakableAmount: stakedAmount,
            });
            await initializeStaker({
                erc20DistributionInstance,
                stakableTokenInstance,
                stakerAddress: secondStakerAddress,
                stakableAmount: stakedAmount,
            });
            await fastForwardTo({
                timestamp: startingTimestamp,
                mineBlockAfter: false,
            });
            // make sure the staking operation happens as soon as possible
            await stakeAtTimestamp(
                erc20DistributionInstance,
                firstStakerAddress,
                stakedAmount,
                startingTimestamp
            );
            const firstStakerStartingTimestamp = await getEvmTimestamp();
            expect(firstStakerStartingTimestamp).to.be.equalBn(
                startingTimestamp
            );
            // make half of the distribution time pass
            await fastForwardTo({
                timestamp: startingTimestamp.add(new BN(5)),
                mineBlockAfter: false,
            });
            await stakeAtTimestamp(
                erc20DistributionInstance,
                secondStakerAddress,
                stakedAmount,
                startingTimestamp.add(new BN(5))
            );
            const secondStakerStartingTimestamp = await getEvmTimestamp();
            expect(secondStakerStartingTimestamp).to.be.equalBn(
                startingTimestamp.add(new BN(5))
            );
            await fastForwardTo({ timestamp: endingTimestamp });
            const onchainStartingTimestamp = await erc20DistributionInstance.startingTimestamp();
            const onchainEndingTimestamp = await erc20DistributionInstance.endingTimestamp();
            expect(onchainStartingTimestamp).to.be.equalBn(startingTimestamp);
            expect(onchainEndingTimestamp).to.be.equalBn(endingTimestamp);
            expect(
                onchainEndingTimestamp.sub(onchainStartingTimestamp)
            ).to.be.equalBn(duration);
            // first staker staked for 10 seconds
            expect(
                onchainEndingTimestamp.sub(firstStakerStartingTimestamp)
            ).to.be.equalBn(new BN(10));
            // second staker staked for 5 seconds
            expect(
                onchainEndingTimestamp.sub(secondStakerStartingTimestamp)
            ).to.be.equalBn(new BN(5));
            const rewardPerSecond = rewardsAmount.div(duration);
            // the first staker had all of the rewards for 5 seconds and half of them for 5
            const expectedFirstStakerReward = rewardPerSecond
                .mul(new BN(5))
                .add(rewardPerSecond.mul(new BN(5)).div(new BN(2)));
            // the second staker had half of the rewards for 5 seconds
            const expectedSecondStakerReward = rewardPerSecond
                .div(new BN(2))
                .mul(new BN(5));
            // first staker claiming/balance checking
            await erc20DistributionInstance.claimAll(firstStakerAddress, {
                from: firstStakerAddress,
            });
            expect(
                await rewardsTokenInstance.balanceOf(firstStakerAddress)
            ).to.be.equalBn(expectedFirstStakerReward);
            // second staker claiming/balance checking
            await erc20DistributionInstance.claimAll(secondStakerAddress, {
                from: secondStakerAddress,
            });
            expect(
                await rewardsTokenInstance.balanceOf(secondStakerAddress)
            ).to.be.equalBn(expectedSecondStakerReward);
        });

        it("should succeed in claiming three rewards if three stakers stake exactly the same amount at different times", async () => {
            const stakedAmount = await toWei(10, stakableTokenInstance);
            const duration = new BN(12);
            const rewardsAmount = await toWei(10, rewardsTokenInstance);
            const {
                startingTimestamp,
                endingTimestamp,
                erc20DistributionInstance,
            } = await initializeDistribution({
                from: ownerAddress,
                erc20DistributionFactoryInstance,
                stakableToken: stakableTokenInstance,
                rewardTokens: [rewardsTokenInstance],
                rewardAmounts: [rewardsAmount],
                duration,
            });
            await initializeStaker({
                erc20DistributionInstance,
                stakableTokenInstance,
                stakerAddress: firstStakerAddress,
                stakableAmount: stakedAmount,
            });
            await initializeStaker({
                erc20DistributionInstance,
                stakableTokenInstance,
                stakerAddress: secondStakerAddress,
                stakableAmount: stakedAmount,
            });
            await initializeStaker({
                erc20DistributionInstance,
                stakableTokenInstance,
                stakerAddress: thirdStakerAddress,
                stakableAmount: stakedAmount,
            });
            await fastForwardTo({
                timestamp: startingTimestamp,
                mineBlockAfter: false,
            });
            // first staker stakes
            await stakeAtTimestamp(
                erc20DistributionInstance,
                firstStakerAddress,
                stakedAmount,
                startingTimestamp
            );
            const firstStakerStartingTimestamp = await getEvmTimestamp();
            expect(firstStakerStartingTimestamp).to.be.equalBn(
                startingTimestamp
            );
            await fastForwardTo({
                timestamp: startingTimestamp.add(new BN(6)),
                mineBlockAfter: false,
            });
            // second staker stakes
            await stakeAtTimestamp(
                erc20DistributionInstance,
                secondStakerAddress,
                stakedAmount,
                startingTimestamp.add(new BN(6))
            );
            const secondStakerStartingTimestamp = await getEvmTimestamp();
            expect(secondStakerStartingTimestamp).to.be.equalBn(
                startingTimestamp.add(new BN(6))
            );
            await fastForwardTo({
                timestamp: secondStakerStartingTimestamp.add(new BN(3)),
                mineBlockAfter: false,
            });
            // third staker stakes
            await stakeAtTimestamp(
                erc20DistributionInstance,
                thirdStakerAddress,
                stakedAmount,
                secondStakerStartingTimestamp.add(new BN(3))
            );
            const thirdStakerStartingTimestamp = await getEvmTimestamp();
            expect(thirdStakerStartingTimestamp).to.be.equalBn(
                secondStakerStartingTimestamp.add(new BN(3))
            );
            // make sure the distribution has ended
            await fastForwardTo({
                timestamp: endingTimestamp.add(new BN(10)),
            });
            const onchainStartingTimestamp = await erc20DistributionInstance.startingTimestamp();
            const onchainEndingTimestamp = await erc20DistributionInstance.endingTimestamp();
            expect(onchainStartingTimestamp).to.be.equalBn(startingTimestamp);
            expect(onchainEndingTimestamp).to.be.equalBn(endingTimestamp);
            expect(
                onchainEndingTimestamp.sub(onchainStartingTimestamp)
            ).to.be.equalBn(duration);

            // first staker staked for 12 seconds
            expect(
                onchainEndingTimestamp.sub(firstStakerStartingTimestamp)
            ).to.be.equalBn(new BN(12));
            // second staker staked for 6 seconds
            expect(
                onchainEndingTimestamp.sub(secondStakerStartingTimestamp)
            ).to.be.equalBn(new BN(6));
            // third staker staked for 3 seconds
            expect(
                onchainEndingTimestamp.sub(thirdStakerStartingTimestamp)
            ).to.be.equalBn(new BN(3));

            // the first staker had all of the rewards for 6 seconds,
            // half of them for 3 seconds and a third for 3 seconds
            const expectedFirstStakerReward = new BN("7083333333333333333");
            // the second staker had half of the rewards for 6 seconds
            // and a third for 3 seconds
            const expectedSecondStakerReward = new BN("2083333333333333333");
            // the third staker had a third of the rewards for 3 seconds
            const expectedThirdStakerReward = new BN("833333333333333333");

            // first staker claiming/balance checking
            await erc20DistributionInstance.claimAll(firstStakerAddress, {
                from: firstStakerAddress,
            });
            expect(
                await rewardsTokenInstance.balanceOf(firstStakerAddress)
            ).to.be.closeBn(expectedFirstStakerReward, MAXIMUM_VARIANCE);

            // second staker claim and rewards balance check
            await erc20DistributionInstance.claimAll(secondStakerAddress, {
                from: secondStakerAddress,
            });
            expect(
                await rewardsTokenInstance.balanceOf(secondStakerAddress)
            ).to.be.closeBn(expectedSecondStakerReward, MAXIMUM_VARIANCE);

            // third staker claim and rewards balance check
            await erc20DistributionInstance.claimAll(thirdStakerAddress, {
                from: thirdStakerAddress,
            });
            expect(
                await rewardsTokenInstance.balanceOf(thirdStakerAddress)
            ).to.be.closeBn(expectedThirdStakerReward, MAXIMUM_VARIANCE);
        });

        it("should succeed in claiming a reward if a staker stakes when the distribution has already started", async () => {
            const stakedAmount = await toWei(10, stakableTokenInstance);
            const duration = new BN(10);
            const rewardsAmount = await toWei(10, rewardsTokenInstance);
            const {
                startingTimestamp,
                endingTimestamp,
                erc20DistributionInstance,
            } = await initializeDistribution({
                from: ownerAddress,
                erc20DistributionFactoryInstance,
                stakableToken: stakableTokenInstance,
                rewardTokens: [rewardsTokenInstance],
                rewardAmounts: [rewardsAmount],
                duration,
            });
            await initializeStaker({
                erc20DistributionInstance,
                stakableTokenInstance,
                stakerAddress: firstStakerAddress,
                stakableAmount: stakedAmount,
            });
            // fast forward to half of the distribution duration
            await fastForwardTo({
                timestamp: startingTimestamp.add(new BN(5)),
                mineBlockAfter: false,
            });
            await stakeAtTimestamp(
                erc20DistributionInstance,
                firstStakerAddress,
                stakedAmount,
                startingTimestamp.add(new BN(5))
            );
            const stakerStartingTimestamp = await getEvmTimestamp();
            expect(stakerStartingTimestamp).to.be.equalBn(
                startingTimestamp.add(new BN(5))
            );
            await fastForwardTo({ timestamp: endingTimestamp });
            const onchainStartingTimestamp = await erc20DistributionInstance.startingTimestamp();
            const onchainEndingTimestamp = await erc20DistributionInstance.endingTimestamp();
            expect(onchainStartingTimestamp).to.be.equalBn(startingTimestamp);
            expect(onchainEndingTimestamp).to.be.equalBn(endingTimestamp);
            // the staker staked for half of the duration
            expect(
                onchainEndingTimestamp.sub(stakerStartingTimestamp)
            ).to.be.equalBn(new BN(5));
            const rewardPerSecond = rewardsAmount.div(duration);
            // the staker had all of the rewards for 5 seconds
            const expectedFirstStakerReward = rewardPerSecond.mul(new BN(5));
            // claim and rewards balance check
            await erc20DistributionInstance.claimAll(firstStakerAddress, {
                from: firstStakerAddress,
            });
            expect(
                await rewardsTokenInstance.balanceOf(firstStakerAddress)
            ).to.be.closeBn(expectedFirstStakerReward, MAXIMUM_VARIANCE);
        });

        it("should fail in claiming 0 rewards if a staker stakes at the last second (literally)", async () => {
            const stakedAmount = await toWei(10, stakableTokenInstance);
            const duration = new BN(10);
            const rewardsAmount = await toWei(10, rewardsTokenInstance);
            const {
                endingTimestamp,
                erc20DistributionInstance,
            } = await initializeDistribution({
                from: ownerAddress,
                erc20DistributionFactoryInstance,
                stakableToken: stakableTokenInstance,
                rewardTokens: [rewardsTokenInstance],
                rewardAmounts: [rewardsAmount],
                duration,
            });
            await initializeStaker({
                erc20DistributionInstance,
                stakableTokenInstance,
                stakerAddress: firstStakerAddress,
                stakableAmount: stakedAmount,
            });
            await fastForwardTo({
                timestamp: endingTimestamp.sub(new BN(1)),
                mineBlockAfter: false,
            });
            const stakerStartingTimestamp = endingTimestamp;
            await stakeAtTimestamp(
                erc20DistributionInstance,
                firstStakerAddress,
                stakedAmount,
                stakerStartingTimestamp
            );
            expect(stakerStartingTimestamp).to.be.equalBn(
                await getEvmTimestamp()
            );
            await fastForwardTo({ timestamp: endingTimestamp });
            const campaignEndingTimestamp = await erc20DistributionInstance.endingTimestamp();
            expect(
                campaignEndingTimestamp.sub(stakerStartingTimestamp)
            ).to.be.equalBn(ZERO_BN);
            try {
                await erc20DistributionInstance.claimAll(firstStakerAddress, {
                    from: firstStakerAddress,
                });
                throw new Error("should have failed");
            } catch (error) {
                expect(error.message).to.contain("SRD23");
            }
        });

        it("should succeed in claiming one rewards if a staker stakes at the last valid distribution second", async () => {
            const stakedAmount = await toWei(10, stakableTokenInstance);
            const duration = new BN(10);
            const rewardsAmount = await toWei(10, rewardsTokenInstance);
            const {
                endingTimestamp,
                erc20DistributionInstance,
            } = await initializeDistribution({
                from: ownerAddress,
                erc20DistributionFactoryInstance,
                stakableToken: stakableTokenInstance,
                rewardTokens: [rewardsTokenInstance],
                rewardAmounts: [rewardsAmount],
                duration,
            });
            await initializeStaker({
                erc20DistributionInstance,
                stakableTokenInstance,
                stakerAddress: firstStakerAddress,
                stakableAmount: stakedAmount,
            });
            await fastForwardTo({
                timestamp: endingTimestamp.sub(new BN(1)),
                mineBlockAfter: false,
            });
            const stakerStartingTimestamp = endingTimestamp.sub(new BN(1));
            await stakeAtTimestamp(
                erc20DistributionInstance,
                firstStakerAddress,
                stakedAmount,
                stakerStartingTimestamp
            );
            expect(stakerStartingTimestamp).to.be.equalBn(
                await getEvmTimestamp()
            );
            await fastForwardTo({ timestamp: endingTimestamp });
            const campaignEndingTimestamp = await erc20DistributionInstance.endingTimestamp();
            expect(
                campaignEndingTimestamp.sub(stakerStartingTimestamp)
            ).to.be.equalBn(new BN(1));
            const rewardPerSecond = rewardsAmount.div(duration);
            await erc20DistributionInstance.claimAll(firstStakerAddress, {
                from: firstStakerAddress,
            });
            expect(
                await rewardsTokenInstance.balanceOf(firstStakerAddress)
            ).to.be.closeBn(rewardPerSecond, MAXIMUM_VARIANCE);
        });

        it("should succeed in claiming two rewards if two stakers stake exactly the same amount at different times, and then the first staker withdraws a portion of his stake", async () => {
            const stakedAmount = await toWei(10, stakableTokenInstance);
            const duration = new BN(10);
            const rewardsAmount = await toWei(10, rewardsTokenInstance);
            const {
                startingTimestamp,
                endingTimestamp,
                erc20DistributionInstance,
            } = await initializeDistribution({
                from: ownerAddress,
                erc20DistributionFactoryInstance,
                stakableToken: stakableTokenInstance,
                rewardTokens: [rewardsTokenInstance],
                rewardAmounts: [rewardsAmount],
                duration,
            });
            await initializeStaker({
                erc20DistributionInstance,
                stakableTokenInstance,
                stakerAddress: firstStakerAddress,
                stakableAmount: stakedAmount,
            });
            await initializeStaker({
                erc20DistributionInstance,
                stakableTokenInstance,
                stakerAddress: secondStakerAddress,
                stakableAmount: stakedAmount,
            });
            await fastForwardTo({
                timestamp: startingTimestamp,
                mineBlockAfter: false,
            });
            await stakeAtTimestamp(
                erc20DistributionInstance,
                firstStakerAddress,
                stakedAmount,
                startingTimestamp
            );
            const firstStakerStartingTimestamp = await getEvmTimestamp();
            expect(firstStakerStartingTimestamp).to.be.equalBn(
                startingTimestamp
            );
            await fastForwardTo({
                timestamp: startingTimestamp.add(new BN(5)),
                mineBlockAfter: false,
            });
            await stakeAtTimestamp(
                erc20DistributionInstance,
                secondStakerAddress,
                stakedAmount,
                startingTimestamp.add(new BN(5))
            );
            const secondStakerStartingTimestamp = await getEvmTimestamp();
            expect(secondStakerStartingTimestamp).to.be.equalBn(
                startingTimestamp.add(new BN(5))
            );
            // first staker withdraws at the eight second
            await fastForwardTo({
                timestamp: secondStakerStartingTimestamp.add(new BN(3)),
                mineBlockAfter: false,
            });
            await withdrawAtTimestamp(
                erc20DistributionInstance,
                firstStakerAddress,
                stakedAmount.div(new BN(2)),
                secondStakerStartingTimestamp.add(new BN(3))
            );
            const firstStakerWithdrawTimestamp = await getEvmTimestamp();
            expect(firstStakerWithdrawTimestamp).to.be.equalBn(
                secondStakerStartingTimestamp.add(new BN(3))
            );
            await fastForwardTo({ timestamp: endingTimestamp });
            const onchainStartingTimestamp = await erc20DistributionInstance.startingTimestamp();
            const onchainEndingTimestamp = await erc20DistributionInstance.endingTimestamp();
            expect(onchainEndingTimestamp).to.be.equalBn(endingTimestamp);
            expect(onchainStartingTimestamp).to.be.equalBn(startingTimestamp);
            expect(
                onchainEndingTimestamp.sub(onchainStartingTimestamp)
            ).to.be.equalBn(duration);
            // first staker staked for 10 seconds
            expect(
                onchainEndingTimestamp.sub(firstStakerStartingTimestamp)
            ).to.be.equalBn(new BN(10));
            // first staker withdrew at second 8, 2 seconds before the end
            expect(
                onchainEndingTimestamp.sub(firstStakerWithdrawTimestamp)
            ).to.be.equalBn(new BN(2));
            // second staker staked for 5 seconds
            expect(
                onchainEndingTimestamp.sub(secondStakerStartingTimestamp)
            ).to.be.equalBn(new BN(5));
            const rewardPerSecond = rewardsAmount.div(duration);
            // the first staker had all of the rewards for 5 seconds, half of them for 3, and a third for 2
            const expectedFirstStakerReward = rewardPerSecond
                .mul(new BN(5))
                .add(rewardPerSecond.mul(new BN(3)).div(new BN(2)))
                .add(rewardPerSecond.mul(new BN(2)).div(new BN(3)));
            // the second staker had half of the rewards for 3 seconds and two thirds for 2
            const expectedSecondStakerReward = rewardPerSecond
                .div(new BN(2))
                .mul(new BN(3))
                .add(
                    rewardPerSecond.mul(new BN(2)).mul(new BN(2)).div(new BN(3))
                );
            // first staker claim and rewards balance check
            await erc20DistributionInstance.claimAll(firstStakerAddress, {
                from: firstStakerAddress,
            });
            expect(
                await rewardsTokenInstance.balanceOf(firstStakerAddress)
            ).to.be.closeBn(expectedFirstStakerReward, MAXIMUM_VARIANCE);
            // second staker claim and rewards balance check
            await erc20DistributionInstance.claimAll(secondStakerAddress, {
                from: secondStakerAddress,
            });
            expect(
                await rewardsTokenInstance.balanceOf(secondStakerAddress)
            ).to.be.closeBn(expectedSecondStakerReward, MAXIMUM_VARIANCE);
        });

        it("should succeed in claiming two rewards if two stakers both stake at the last valid distribution second", async () => {
            const stakedAmount = await toWei(10, stakableTokenInstance);
            const duration = new BN(10);
            const rewardsAmount = await toWei(10, rewardsTokenInstance);
            const {
                endingTimestamp,
                erc20DistributionInstance,
            } = await initializeDistribution({
                from: ownerAddress,
                erc20DistributionFactoryInstance,
                stakableToken: stakableTokenInstance,
                rewardTokens: [rewardsTokenInstance],
                rewardAmounts: [rewardsAmount],
                duration,
            });
            await initializeStaker({
                erc20DistributionInstance,
                stakableTokenInstance,
                stakerAddress: firstStakerAddress,
                stakableAmount: stakedAmount,
            });
            await initializeStaker({
                erc20DistributionInstance,
                stakableTokenInstance,
                stakerAddress: secondStakerAddress,
                stakableAmount: stakedAmount,
            });
            await stopMining();
            const stakingTimestamp = endingTimestamp.sub(new BN(1));
            await fastForwardTo({
                timestamp: stakingTimestamp,
                mineBlockAfter: false,
            });
            await stakeAtTimestamp(
                erc20DistributionInstance,
                firstStakerAddress,
                stakedAmount,
                stakingTimestamp
            );
            await stakeAtTimestamp(
                erc20DistributionInstance,
                secondStakerAddress,
                stakedAmount,
                stakingTimestamp
            );
            expect(await getEvmTimestamp()).to.be.equalBn(stakingTimestamp);
            await startMining();
            await fastForwardTo({ timestamp: endingTimestamp });

            const onchainEndingTimestamp = await erc20DistributionInstance.endingTimestamp();
            const onchainStartingTimestamp = await erc20DistributionInstance.startingTimestamp();
            expect(
                onchainEndingTimestamp.sub(onchainStartingTimestamp)
            ).to.be.equalBn(duration);
            expect(onchainEndingTimestamp.sub(stakingTimestamp)).to.be.equalBn(
                new BN(1)
            );

            const rewardPerSecond = rewardsAmount.div(duration);
            // the first staker had half of the rewards for 1 second
            const expectedFirstStakerReward = rewardPerSecond.div(new BN(2));
            // the second staker had half of the rewards for 1 second
            const expectedSecondStakerReward = rewardPerSecond.div(new BN(2));

            await erc20DistributionInstance.claimAll(firstStakerAddress, {
                from: firstStakerAddress,
            });
            expect(
                await rewardsTokenInstance.balanceOf(firstStakerAddress)
            ).to.be.closeBn(expectedFirstStakerReward, MAXIMUM_VARIANCE);

            // second staker claim and rewards balance check
            await erc20DistributionInstance.claimAll(secondStakerAddress, {
                from: secondStakerAddress,
            });
            expect(
                await rewardsTokenInstance.balanceOf(secondStakerAddress)
            ).to.be.closeBn(expectedSecondStakerReward, MAXIMUM_VARIANCE);
        });

        it("should succeed in claiming a reward if a staker stakes at second n and then increases their stake", async () => {
            const stakedAmount = await toWei(10, stakableTokenInstance);
            const duration = new BN(10);
            const rewardsAmount = await toWei(10, rewardsTokenInstance);
            const amountPerStake = stakedAmount.div(new BN(2));
            const {
                startingTimestamp,
                endingTimestamp,
                erc20DistributionInstance,
            } = await initializeDistribution({
                from: ownerAddress,
                erc20DistributionFactoryInstance,
                stakableToken: stakableTokenInstance,
                rewardTokens: [rewardsTokenInstance],
                rewardAmounts: [rewardsAmount],
                duration,
            });
            await initializeStaker({
                erc20DistributionInstance,
                stakableTokenInstance,
                stakerAddress: firstStakerAddress,
                stakableAmount: stakedAmount,
            });
            await fastForwardTo({
                timestamp: startingTimestamp,
                mineBlockAfter: false,
            });
            await stakeAtTimestamp(
                erc20DistributionInstance,
                firstStakerAddress,
                amountPerStake,
                startingTimestamp
            );
            const firstStakeStartingTimestamp = await getEvmTimestamp();
            expect(firstStakeStartingTimestamp).to.be.equalBn(
                startingTimestamp
            );
            await fastForwardTo({
                timestamp: startingTimestamp.add(new BN(5)),
                mineBlockAfter: false,
            });
            await stakeAtTimestamp(
                erc20DistributionInstance,
                firstStakerAddress,
                amountPerStake,
                startingTimestamp.add(new BN(5))
            );
            const secondStakeStartingTimestamp = await getEvmTimestamp();
            await fastForwardTo({ timestamp: endingTimestamp });
            const onchainEndingTimestamp = await erc20DistributionInstance.endingTimestamp();
            const onchainStartingTimestamp = await erc20DistributionInstance.startingTimestamp();
            expect(
                onchainEndingTimestamp.sub(onchainStartingTimestamp)
            ).to.be.equalBn(duration);
            expect(
                onchainEndingTimestamp.sub(firstStakeStartingTimestamp)
            ).to.be.equalBn(new BN(10));
            expect(
                onchainEndingTimestamp.sub(secondStakeStartingTimestamp)
            ).to.be.equalBn(new BN(5));
            await erc20DistributionInstance.claimAll(firstStakerAddress, {
                from: firstStakerAddress,
            });
            expect(
                await rewardsTokenInstance.balanceOf(firstStakerAddress)
            ).to.be.equalBn(rewardsAmount);
        });

        it("should succeed in claiming two rewards if two staker respectively stake and withdraw at the same second", async () => {
            const stakedAmount = await toWei(10, stakableTokenInstance);
            const duration = new BN(10);
            const rewardsAmount = await toWei(10, rewardsTokenInstance);
            const {
                startingTimestamp,
                endingTimestamp,
                erc20DistributionInstance,
            } = await initializeDistribution({
                from: ownerAddress,
                erc20DistributionFactoryInstance,
                stakableToken: stakableTokenInstance,
                rewardTokens: [rewardsTokenInstance],
                rewardAmounts: [rewardsAmount],
                duration,
            });
            await initializeStaker({
                erc20DistributionInstance,
                stakableTokenInstance,
                stakerAddress: firstStakerAddress,
                stakableAmount: stakedAmount,
            });
            await initializeStaker({
                erc20DistributionInstance,
                stakableTokenInstance,
                stakerAddress: secondStakerAddress,
                stakableAmount: stakedAmount,
            });
            await fastForwardTo({ timestamp: startingTimestamp });
            await stakeAtTimestamp(
                erc20DistributionInstance,
                firstStakerAddress,
                stakedAmount,
                startingTimestamp
            );
            const firstStakerStartingTimestamp = await getEvmTimestamp();
            expect(firstStakerStartingTimestamp).to.be.equalBn(
                startingTimestamp
            );
            await stopMining();
            const stakeAndWithdrawTimestamp = startingTimestamp.add(new BN(5));
            await fastForwardTo({
                timestamp: stakeAndWithdrawTimestamp,
                mineBlockAfter: false,
            });
            await stakeAtTimestamp(
                erc20DistributionInstance,
                secondStakerAddress,
                stakedAmount,
                stakeAndWithdrawTimestamp
            );
            await withdrawAtTimestamp(
                erc20DistributionInstance,
                firstStakerAddress,
                stakedAmount,
                stakeAndWithdrawTimestamp
            );
            const secondStakerStartingTimestamp = await getEvmTimestamp();
            const firstStakerWithdrawTimestamp = await getEvmTimestamp();
            await startMining();
            expect(secondStakerStartingTimestamp).to.be.equalBn(
                stakeAndWithdrawTimestamp
            );
            expect(firstStakerWithdrawTimestamp).to.be.equalBn(
                stakeAndWithdrawTimestamp
            );
            await fastForwardTo({ timestamp: endingTimestamp });
            const onchainEndingTimestamp = await erc20DistributionInstance.endingTimestamp();
            const onchainStartingTimestamp = await erc20DistributionInstance.startingTimestamp();
            expect(
                onchainEndingTimestamp.sub(onchainStartingTimestamp)
            ).to.be.equalBn(duration);
            expect(
                firstStakerWithdrawTimestamp.sub(firstStakerStartingTimestamp)
            ).to.be.equalBn(new BN(5));
            expect(
                onchainEndingTimestamp.sub(secondStakerStartingTimestamp)
            ).to.be.equalBn(new BN(5));

            const rewardPerSecond = rewardsAmount.div(duration);
            // both stakers had all of the rewards for 5 seconds
            const expectedReward = rewardPerSecond.mul(new BN(5));

            // first staker claim and rewards balance check
            await erc20DistributionInstance.claimAll(firstStakerAddress, {
                from: firstStakerAddress,
            });
            expect(
                await rewardsTokenInstance.balanceOf(firstStakerAddress)
            ).to.be.equalBn(expectedReward);

            // second staker claim and rewards balance check
            await erc20DistributionInstance.claimAll(secondStakerAddress, {
                from: secondStakerAddress,
            });
            expect(
                await rewardsTokenInstance.balanceOf(secondStakerAddress)
            ).to.be.equalBn(expectedReward);
        });

        it("should succeed when staggered operations happen (test that found a previous bug)", async () => {
            // what happens here:
            // - First staker stakes
            // - Second staker stakes
            // - First staker fully withdraws
            // - Second staker fully withdraws (no more staked tokens in the contract)
            // - First staker claims all
            // - Second staker claims all
            // - First staker restakes right in the ending 2 seconds
            // - First staker claims accrued rewards after the campaign ended

            const stakedAmount = await toWei(10, stakableTokenInstance);
            const {
                startingTimestamp,
                endingTimestamp,
                erc20DistributionInstance,
            } = await initializeDistribution({
                from: ownerAddress,
                erc20DistributionFactoryInstance,
                stakableToken: stakableTokenInstance,
                rewardTokens: [rewardsTokenInstance],
                rewardAmounts: [await toWei(10, rewardsTokenInstance)],
                duration: 10,
                stakingCap: 0,
            });
            await initializeStaker({
                erc20DistributionInstance,
                stakableTokenInstance,
                stakerAddress: firstStakerAddress,
                stakableAmount: stakedAmount.mul(new BN(2)),
            });
            await initializeStaker({
                erc20DistributionInstance,
                stakableTokenInstance,
                stakerAddress: secondStakerAddress,
                stakableAmount: stakedAmount,
            });

            // first staker stakes at the start
            await fastForwardTo({ timestamp: startingTimestamp });
            await stakeAtTimestamp(
                erc20DistributionInstance,
                firstStakerAddress,
                stakedAmount,
                startingTimestamp
            );

            // second staker stakes at 3 seconds
            const secondStakingTimestamp = startingTimestamp.add(new BN(3));
            await fastForwardTo({ timestamp: secondStakingTimestamp });
            await stakeAtTimestamp(
                erc20DistributionInstance,
                secondStakerAddress,
                stakedAmount,
                secondStakingTimestamp
            );

            // first staker withdraws at 5 seconds
            const firstWithdrawingTimestamp = secondStakingTimestamp.add(
                new BN(2)
            );
            await fastForwardTo({ timestamp: firstWithdrawingTimestamp });
            await withdrawAtTimestamp(
                erc20DistributionInstance,
                firstStakerAddress,
                stakedAmount,
                firstWithdrawingTimestamp
            );

            // second staker withdraws at 6 seconds
            const secondWithdrawingTimestamp = firstWithdrawingTimestamp.add(
                new BN(1)
            );
            await fastForwardTo({ timestamp: secondWithdrawingTimestamp });
            await withdrawAtTimestamp(
                erc20DistributionInstance,
                secondStakerAddress,
                stakedAmount,
                secondWithdrawingTimestamp
            );

            // first staker claims reward and at stakes at 8 seconds
            await stopMining();
            const firstClaimAndRestakeTimestamp = secondWithdrawingTimestamp.add(
                new BN(2)
            );
            await fastForwardTo({
                timestamp: firstClaimAndRestakeTimestamp,
                mineBlockAfter: false,
            });
            await claimAllAtTimestamp(
                erc20DistributionInstance,
                firstStakerAddress,
                firstStakerAddress,
                firstClaimAndRestakeTimestamp
            );
            await stakeAtTimestamp(
                erc20DistributionInstance,
                firstStakerAddress,
                stakedAmount,
                firstClaimAndRestakeTimestamp
            );
            expect(await getEvmTimestamp()).to.be.equalBn(
                firstClaimAndRestakeTimestamp
            );
            await startMining();

            // second staker now claims their previously accrued rewards. With the found and now fixed bug, this
            // would have reverted due to the fact that when the first staker claimed, the reward per staked token for
            // each reward token was put to 0, alongside the consolidated reward per staked token FOR THE FIRST STAKER ONLY.
            // Issue is that the consolidated reward per staked token of the second staker wasn't put to zero.
            // When then consolidating the reward in the last consolidation period for the second staker, when claiming
            // their reward in the following instruction, a calculation was made:
            // `reward.perStakedToken - staker.consolidatedRewardPerStakedToken[reward.token]`, to account for the last
            // consolidation checkpointing. In this scenario, reward.perStakedToken was zero,
            // while the consolidated amount wasn't. This caused an underflow, which now reverts in Solidity 0.8.0.
            const secondClaimTimestamp = firstClaimAndRestakeTimestamp.add(
                new BN(1)
            );
            await fastForwardTo({
                timestamp: secondClaimTimestamp,
            });
            await claimAllAtTimestamp(
                erc20DistributionInstance,
                secondStakerAddress,
                secondStakerAddress,
                secondClaimTimestamp
            );

            // fast forwarding to the end of the campaign
            await fastForwardTo({
                timestamp: endingTimestamp,
            });

            // first staker staked at the start for 5 seconds, while the second staked at 3 seconds
            // for 3 seconds. The two stakers overlapped by a grand total of 2 seconds.
            // The first staker then staked again in the last 2 seconds, but we'll account for
            // this and claim these rewards later in the test.

            // First staker got full rewards for 3 seconds and half rewards for 2 seconds. At a rate
            // of 1 reward token/second, this translates to a reward of 3 + (0.5 * 2) = 4
            const expectedFirstStakerReward = new BN(
                await toWei(4, rewardsTokenInstance)
            );

            // Second staker got full rewards for 1 second and half rewards for 2 seconds. At a rate
            // of 1 reward token/second, this translates to a reward of 1 + (0.5 * 2) = 2
            const expectedSecondStakerReward = new BN(
                await toWei(2, rewardsTokenInstance)
            );

            expect(
                await rewardsTokenInstance.balanceOf(firstStakerAddress)
            ).to.be.closeBn(expectedFirstStakerReward, MAXIMUM_VARIANCE);
            expect(
                await rewardsTokenInstance.balanceOf(secondStakerAddress)
            ).to.be.closeBn(expectedSecondStakerReward, MAXIMUM_VARIANCE);

            // used to see how much stuff was actually claimed in the second claim
            const preClaimBalance = await rewardsTokenInstance.balanceOf(
                firstStakerAddress
            );
            // now claiming the remaining rewards for the first staker (mentioned in the comment above)
            await erc20DistributionInstance.claimAll(firstStakerAddress, {
                from: firstStakerAddress,
            });
            // the first staker staked at the end for 2 seconds. At a reward rate of 1 token/second,
            // 2 reward tokens are expected to be claimed
            const postClaimBalance = await rewardsTokenInstance.balanceOf(
                firstStakerAddress
            );
            const expectedRemainingReward = await toWei(
                2,
                rewardsTokenInstance
            );
            expect(postClaimBalance.sub(preClaimBalance)).to.be.closeBn(
                expectedRemainingReward,
                MAXIMUM_VARIANCE
            );

            // we also test recovery for good measure. There have been staked tokens in the contract
            // for all but 2 seconds (first staker staked at the start for 5 seconds and second staker
            // staked at second 3 for 3 seconds, overlapping for 2, and then first staker restaked
            // at the 8th second until the end)
            await erc20DistributionInstance.recoverUnassignedRewards({
                from: ownerAddress,
            });
            const expectedRecoveredReward = await toWei(
                2,
                rewardsTokenInstance
            );
            expect(
                await rewardsTokenInstance.balanceOf(ownerAddress)
            ).to.be.closeBn(expectedRecoveredReward, MAXIMUM_VARIANCE);

            // At this point all the tokens minus some wei due to integer truncation should
            // have been recovered from the contract.
            // Initial reward was 10 tokens, the first staker got 6 in total, the second staker
            // 2, and the owner recovered 2.
        });
    }
);
```


#### test/erc20-distribution/single-tokens/initialization.test.js
```javascript
const BN = require("bn.js");
const { expect } = require("chai");
const { ZERO_ADDRESS } = require("../../constants");
const { initializeDistribution } = require("../../utils");
const { toWei } = require("../../utils/conversion");
const { getEvmTimestamp, fastForwardTo } = require("../../utils/network");

const ERC20StakingRewardsDistribution = artifacts.require(
    "ERC20StakingRewardsDistribution"
);
const ERC20StakingRewardsDistributionFactory = artifacts.require(
    "ERC20StakingRewardsDistributionFactory"
);
const FirstRewardERC20 = artifacts.require("FirstRewardERC20");
const FirstStakableERC20 = artifacts.require("FirstStakableERC20");

contract(
    "ERC20StakingRewardsDistribution - Single reward/stakable token - Initialization",
    () => {
        let erc20DistributionFactoryInstance,
            rewardsTokenInstance,
            stakableTokenInstance,
            ownerAddress;

        beforeEach(async () => {
            const accounts = await web3.eth.getAccounts();
            ownerAddress = accounts[0];
            const erc20DistributionInstance = await ERC20StakingRewardsDistribution.new(
                { from: ownerAddress }
            );
            erc20DistributionFactoryInstance = await ERC20StakingRewardsDistributionFactory.new(
                erc20DistributionInstance.address,
                { from: ownerAddress }
            );
            rewardsTokenInstance = await FirstRewardERC20.new();
            stakableTokenInstance = await FirstStakableERC20.new();
        });

        it("should fail when passing a 0-address rewards token", async () => {
            try {
                const erc20DistributionInstance = await ERC20StakingRewardsDistribution.new(
                    { from: ownerAddress }
                );
                await erc20DistributionInstance.initialize(
                    [ZERO_ADDRESS],
                    stakableTokenInstance.address,
                    [10],
                    10000000000,
                    100000000000,
                    false,
                    0
                );
                throw new Error("should have failed");
            } catch (error) {
                expect(error.message).to.contain("SRD04");
            }
        });

        it("should fail when passing a 0-address stakable token", async () => {
            try {
                await initializeDistribution({
                    from: ownerAddress,
                    erc20DistributionFactoryInstance,
                    stakableToken: { address: ZERO_ADDRESS },
                    rewardTokens: [rewardsTokenInstance],
                    rewardAmounts: [14],
                    duration: 10,
                });
                throw new Error("should have failed");
            } catch (error) {
                expect(error.message).to.contain("SRD07");
            }
        });

        it("should fail when passing 0 as a rewards amount", async () => {
            try {
                await initializeDistribution({
                    from: ownerAddress,
                    erc20DistributionFactoryInstance,
                    stakableToken: stakableTokenInstance,
                    rewardTokens: [rewardsTokenInstance],
                    rewardAmounts: [0],
                    duration: 10,
                });
                throw new Error("should have failed");
            } catch (error) {
                expect(error.message).to.contain("SRD05");
            }
        });

        it("should fail when passing a lower starting timestamp than the current one", async () => {
            try {
                const currentEvmTimestamp = await getEvmTimestamp();
                const erc20DistributionInstance = await ERC20StakingRewardsDistribution.new(
                    { from: ownerAddress }
                );
                await erc20DistributionInstance.initialize(
                    [rewardsTokenInstance.address],
                    stakableTokenInstance.address,
                    [1],
                    currentEvmTimestamp.sub(new BN(10)),
                    currentEvmTimestamp.add(new BN(10)),
                    false,
                    0,
                    { from: ownerAddress }
                );
                throw new Error("should have failed");
            } catch (error) {
                expect(error.message).to.contain("SRD01");
            }
        });

        it("should fail when passing the same starting timestamp as the current one", async () => {
            try {
                const currentEvmTimestamp = await getEvmTimestamp();
                const erc20DistributionInstance = await ERC20StakingRewardsDistribution.new(
                    { from: ownerAddress }
                );
                await erc20DistributionInstance.initialize(
                    [rewardsTokenInstance.address],
                    stakableTokenInstance.address,
                    [1],
                    currentEvmTimestamp,
                    currentEvmTimestamp.add(new BN(10)),
                    false,
                    0,
                    { from: ownerAddress }
                );
                throw new Error("should have failed");
            } catch (error) {
                expect(error.message).to.contain("SRD01");
            }
        });

        it("should fail when passing 0 as seconds duration", async () => {
            try {
                await initializeDistribution({
                    from: ownerAddress,
                    erc20DistributionFactoryInstance,
                    stakableToken: stakableTokenInstance,
                    rewardTokens: [rewardsTokenInstance],
                    rewardAmounts: [1],
                    duration: 0,
                });
                throw new Error("should have failed");
            } catch (error) {
                expect(error.message).to.contain("SRD02");
            }
        });

        it("should succeed in the right conditions", async () => {
            const rewardAmounts = [
                new BN(await toWei(10, rewardsTokenInstance)),
            ];
            const duration = new BN(10);
            const rewardTokens = [rewardsTokenInstance];
            const {
                startingTimestamp,
                endingTimestamp,
                erc20DistributionInstance,
            } = await initializeDistribution({
                from: ownerAddress,
                erc20DistributionFactoryInstance,
                stakableToken: stakableTokenInstance,
                rewardTokens,
                rewardAmounts,
                duration,
            });
            await fastForwardTo(startingTimestamp);

            expect(await erc20DistributionInstance.initialized()).to.be.true;
            const onchainRewardTokens = await erc20DistributionInstance.getRewardTokens();
            expect(onchainRewardTokens).to.have.length(rewardTokens.length);
            expect(onchainRewardTokens[0]).to.be.equal(
                rewardsTokenInstance.address
            );
            expect(await erc20DistributionInstance.stakableToken()).to.be.equal(
                stakableTokenInstance.address
            );
            for (let i = 0; i < rewardTokens.length; i++) {
                const rewardAmount = rewardAmounts[i];
                const rewardToken = rewardTokens[i];
                expect(
                    await rewardToken.balanceOf(
                        erc20DistributionInstance.address
                    )
                ).to.be.equalBn(rewardAmount);
                expect(
                    await erc20DistributionInstance.rewardAmount(
                        rewardToken.address
                    )
                ).to.be.equalBn(rewardAmount);
            }
            const onchainStartingTimestamp = await erc20DistributionInstance.startingTimestamp();
            expect(onchainStartingTimestamp).to.be.equalBn(startingTimestamp);
            const onchainEndingTimestamp = await erc20DistributionInstance.endingTimestamp();
            expect(onchainEndingTimestamp.sub(startingTimestamp)).to.be.equalBn(
                duration
            );
            expect(onchainEndingTimestamp).to.be.equalBn(endingTimestamp);
        });

        it("should fail when trying to initialize a second time", async () => {
            try {
                const {
                    erc20DistributionInstance,
                } = await initializeDistribution({
                    from: ownerAddress,
                    erc20DistributionFactoryInstance,
                    stakableToken: stakableTokenInstance,
                    rewardTokens: [rewardsTokenInstance],
                    rewardAmounts: [2],
                    duration: 2,
                });
                await erc20DistributionInstance.initialize(
                    [rewardsTokenInstance.address],
                    stakableTokenInstance.address,
                    [7],
                    1000000000000,
                    10000000000000,
                    false,
                    0
                );
                throw new Error("should have failed");
            } catch (error) {
                expect(error.message).to.contain("SRD18");
            }
        });
    }
);
```
#### test/erc20-distribution/single-tokens/recovering.test.js
```javascript
const BN = require("bn.js");
const { expect } = require("chai");
const { ZERO_BN } = require("../../constants");
const {
    initializeDistribution,
    initializeStaker,
    withdrawAtTimestamp,
    stakeAtTimestamp,
    claimAllAtTimestamp,
    recoverUnassignedRewardsAtTimestamp,
} = require("../../utils");
const { toWei } = require("../../utils/conversion");
const {
    stopMining,
    startMining,
    fastForwardTo,
    getEvmTimestamp,
} = require("../../utils/network");

const ERC20StakingRewardsDistribution = artifacts.require(
    "ERC20StakingRewardsDistribution"
);
const ERC20StakingRewardsDistributionFactory = artifacts.require(
    "ERC20StakingRewardsDistributionFactory"
);
const FirstRewardERC20 = artifacts.require("FirstRewardERC20");
const FirstStakableERC20 = artifacts.require("FirstStakableERC20");

contract(
    "ERC20StakingRewardsDistribution - Single reward/stakable token - Reward recovery",
    () => {
        let erc20DistributionFactoryInstance,
            rewardsTokenInstance,
            stakableTokenInstance,
            ownerAddress,
            firstStakerAddress,
            secondStakerAddress;

        beforeEach(async () => {
            const accounts = await web3.eth.getAccounts();
            ownerAddress = accounts[0];
            const erc20DistributionInstance = await ERC20StakingRewardsDistribution.new(
                { from: ownerAddress }
            );
            erc20DistributionFactoryInstance = await ERC20StakingRewardsDistributionFactory.new(
                erc20DistributionInstance.address,
                { from: ownerAddress }
            );
            rewardsTokenInstance = await FirstRewardERC20.new();
            stakableTokenInstance = await FirstStakableERC20.new();
            firstStakerAddress = accounts[1];
            secondStakerAddress = accounts[2];
        });

        it("should fail when the distribution is not initialized", async () => {
            try {
                const erc20DistributionInstance = await ERC20StakingRewardsDistribution.new(
                    { from: ownerAddress }
                );
                await erc20DistributionInstance.recoverUnassignedRewards();
                throw new Error("should have failed");
            } catch (error) {
                expect(error.message).to.contain("SRD20");
            }
        });

        it("should fail when the distribution has not yet started", async () => {
            try {
                const {
                    erc20DistributionInstance,
                } = await initializeDistribution({
                    from: ownerAddress,
                    erc20DistributionFactoryInstance,
                    stakableToken: stakableTokenInstance,
                    rewardTokens: [rewardsTokenInstance],
                    rewardAmounts: [11],
                    duration: 10,
                });
                await erc20DistributionInstance.recoverUnassignedRewards();
                throw new Error("should have failed");
            } catch (error) {
                expect(error.message).to.contain("SRD20");
            }
        });

        it("should recover all of the rewards when the distribution ended and no staker joined", async () => {
            const rewardsAmount = await toWei(100, rewardsTokenInstance);
            const {
                endingTimestamp,
                erc20DistributionInstance,
            } = await initializeDistribution({
                from: ownerAddress,
                erc20DistributionFactoryInstance,
                stakableToken: stakableTokenInstance,
                rewardTokens: [rewardsTokenInstance],
                rewardAmounts: [rewardsAmount],
                duration: 10,
            });
            // at the start of the distribution, the owner deposited the reward
            // into the staking contract, so theur balance is 0
            expect(
                await rewardsTokenInstance.balanceOf(ownerAddress)
            ).to.be.equalBn(ZERO_BN);
            await fastForwardTo({ timestamp: endingTimestamp });
            await erc20DistributionInstance.recoverUnassignedRewards();
            expect(
                await rewardsTokenInstance.balanceOf(ownerAddress)
            ).to.be.equalBn(rewardsAmount);
        });

        it("should put the recoverable rewards variable to 0 when recovered", async () => {
            const rewardsAmount = await toWei(100, rewardsTokenInstance);
            const {
                endingTimestamp,
                erc20DistributionInstance,
            } = await initializeDistribution({
                from: ownerAddress,
                erc20DistributionFactoryInstance,
                stakableToken: stakableTokenInstance,
                rewardTokens: [rewardsTokenInstance],
                rewardAmounts: [rewardsAmount],
                duration: 10,
            });
            await fastForwardTo({ timestamp: endingTimestamp });
            await erc20DistributionInstance.recoverUnassignedRewards();
            expect(
                await rewardsTokenInstance.balanceOf(ownerAddress)
            ).to.be.equalBn(rewardsAmount);
            expect(
                await erc20DistributionInstance.recoverableUnassignedReward(
                    rewardsTokenInstance.address
                )
            ).to.be.equalBn(ZERO_BN);
        });

        it("should always send funds to the contract's owner, even when called by another account", async () => {
            const rewardsAmount = await toWei(100, rewardsTokenInstance);
            const {
                endingTimestamp,
                erc20DistributionInstance,
            } = await initializeDistribution({
                from: ownerAddress,
                erc20DistributionFactoryInstance,
                stakableToken: stakableTokenInstance,
                rewardTokens: [rewardsTokenInstance],
                rewardAmounts: [rewardsAmount],
                duration: 10,
            });
            // at the start of the distribution, the owner deposited the reward
            // into the staking contract, so theur balance is 0
            expect(
                await rewardsTokenInstance.balanceOf(ownerAddress)
            ).to.be.equalBn(ZERO_BN);
            await fastForwardTo({ timestamp: endingTimestamp });
            await erc20DistributionInstance.recoverUnassignedRewards({
                from: secondStakerAddress,
            });
            expect(
                await rewardsTokenInstance.balanceOf(secondStakerAddress)
            ).to.be.equalBn(ZERO_BN);
            expect(
                await rewardsTokenInstance.balanceOf(ownerAddress)
            ).to.be.equalBn(rewardsAmount);
        });

        it("should recover half of the rewards when only one staker joined for half of the duration", async () => {
            const rewardsAmount = await toWei(100, rewardsTokenInstance);
            const {
                startingTimestamp,
                endingTimestamp,
                erc20DistributionInstance,
            } = await initializeDistribution({
                from: ownerAddress,
                erc20DistributionFactoryInstance,
                stakableToken: stakableTokenInstance,
                rewardTokens: [rewardsTokenInstance],
                rewardAmounts: [rewardsAmount],
                duration: 10,
            });
            await initializeStaker({
                erc20DistributionInstance,
                stakableTokenInstance,
                stakerAddress: firstStakerAddress,
                stakableAmount: 1,
            });
            expect(
                await rewardsTokenInstance.balanceOf(ownerAddress)
            ).to.be.equalBn(ZERO_BN);
            // stake after 5 seconds until the end of the distribution
            const stakingTimestamp = startingTimestamp.add(new BN(5));
            await fastForwardTo({ timestamp: stakingTimestamp });
            await stakeAtTimestamp(
                erc20DistributionInstance,
                firstStakerAddress,
                [1],
                stakingTimestamp
            );
            await fastForwardTo({ timestamp: endingTimestamp });
            const onchainEndingTimestamp = await erc20DistributionInstance.endingTimestamp();
            // staker staked for 5 seconds
            expect(onchainEndingTimestamp.sub(stakingTimestamp)).to.be.equalBn(
                new BN(5)
            );
            // staker claims their reward
            await erc20DistributionInstance.claimAll(firstStakerAddress, {
                from: firstStakerAddress,
            });
            expect(
                await rewardsTokenInstance.balanceOf(firstStakerAddress)
            ).to.be.equalBn(await toWei("50", rewardsTokenInstance));
            await erc20DistributionInstance.recoverUnassignedRewards();
            expect(
                await rewardsTokenInstance.balanceOf(ownerAddress)
            ).to.be.equalBn(await toWei("50", rewardsTokenInstance));
        });

        it("should recover half of the rewards when two stakers stake the same time", async () => {
            const rewardsAmount = await toWei(100, rewardsTokenInstance);
            const {
                startingTimestamp,
                endingTimestamp,
                erc20DistributionInstance,
            } = await initializeDistribution({
                from: ownerAddress,
                erc20DistributionFactoryInstance,
                stakableToken: stakableTokenInstance,
                rewardTokens: [rewardsTokenInstance],
                rewardAmounts: [rewardsAmount],
                duration: 10,
            });
            await initializeStaker({
                erc20DistributionInstance,
                stakableTokenInstance,
                stakerAddress: firstStakerAddress,
                stakableAmount: 1,
            });
            await initializeStaker({
                erc20DistributionInstance,
                stakableTokenInstance,
                stakerAddress: secondStakerAddress,
                stakableAmount: 1,
            });
            expect(
                await rewardsTokenInstance.balanceOf(ownerAddress)
            ).to.be.equalBn(ZERO_BN);
            // stake after 5 seconds until the end of the distribution
            await stopMining();
            const stakingTimestamp = startingTimestamp.add(new BN(5));
            await fastForwardTo({ timestamp: stakingTimestamp });
            await stakeAtTimestamp(
                erc20DistributionInstance,
                firstStakerAddress,
                [1],
                stakingTimestamp
            );
            await stakeAtTimestamp(
                erc20DistributionInstance,
                secondStakerAddress,
                [1],
                stakingTimestamp
            );
            expect(await getEvmTimestamp()).to.be.equalBn(stakingTimestamp);
            await startMining();
            await fastForwardTo({ timestamp: endingTimestamp });
            const distributionEndingTimestamp = await erc20DistributionInstance.endingTimestamp();
            // each staker staked for 5 seconds
            expect(
                distributionEndingTimestamp.sub(stakingTimestamp)
            ).to.be.equalBn(new BN(5));
            // stakers claim their reward
            const expectedReward = await toWei("25", rewardsTokenInstance);
            await erc20DistributionInstance.claimAll(firstStakerAddress, {
                from: firstStakerAddress,
            });
            expect(
                await rewardsTokenInstance.balanceOf(firstStakerAddress)
            ).to.be.equalBn(expectedReward);
            await erc20DistributionInstance.claimAll(secondStakerAddress, {
                from: secondStakerAddress,
            });
            expect(
                await rewardsTokenInstance.balanceOf(secondStakerAddress)
            ).to.be.equalBn(expectedReward);
            await erc20DistributionInstance.recoverUnassignedRewards();
            expect(
                await rewardsTokenInstance.balanceOf(ownerAddress)
            ).to.be.equalBn(rewardsAmount.div(new BN(2)));
        });

        it("should recover a third of the rewards when a staker stakes for two thirds of the distribution duration", async () => {
            const rewardsAmount = await toWei(100, rewardsTokenInstance);
            const {
                startingTimestamp,
                endingTimestamp,
                erc20DistributionInstance,
            } = await initializeDistribution({
                from: ownerAddress,
                erc20DistributionFactoryInstance,
                stakableToken: stakableTokenInstance,
                rewardTokens: [rewardsTokenInstance],
                rewardAmounts: [rewardsAmount],
                duration: 12,
            });
            await initializeStaker({
                erc20DistributionInstance,
                stakableTokenInstance,
                stakerAddress: firstStakerAddress,
                stakableAmount: 1,
            });
            expect(
                await rewardsTokenInstance.balanceOf(ownerAddress)
            ).to.be.equalBn(ZERO_BN);
            // stake after 4 seconds until the end of the distribution
            const stakingTimestamp = startingTimestamp.add(new BN(4));
            await fastForwardTo({ timestamp: stakingTimestamp });
            await stakeAtTimestamp(
                erc20DistributionInstance,
                firstStakerAddress,
                [1],
                stakingTimestamp
            );
            await fastForwardTo({ timestamp: endingTimestamp });
            const onchainEndingTimestamp = await erc20DistributionInstance.endingTimestamp();
            expect(onchainEndingTimestamp.sub(stakingTimestamp)).to.be.equalBn(
                new BN(8)
            );
            // staker claims their reward
            const expectedReward = new BN("66666666666666666666");
            await erc20DistributionInstance.claimAll(firstStakerAddress, {
                from: firstStakerAddress,
            });
            expect(
                await rewardsTokenInstance.balanceOf(firstStakerAddress)
            ).to.be.equalBn(expectedReward);
            await erc20DistributionInstance.recoverUnassignedRewards();
            expect(
                await rewardsTokenInstance.balanceOf(ownerAddress)
            ).to.be.equalBn(new BN("33333333333333333333"));
        });

        it("should recover two thirds of the rewards when a staker stakes for a third of the distribution duration, right in the middle", async () => {
            const rewardsAmount = await toWei(100, rewardsTokenInstance);
            const {
                startingTimestamp,
                endingTimestamp,
                erc20DistributionInstance,
            } = await initializeDistribution({
                from: ownerAddress,
                erc20DistributionFactoryInstance,
                stakableToken: stakableTokenInstance,
                rewardTokens: [rewardsTokenInstance],
                rewardAmounts: [rewardsAmount],
                duration: 12,
            });
            await initializeStaker({
                erc20DistributionInstance,
                stakableTokenInstance,
                stakerAddress: firstStakerAddress,
                stakableAmount: 1,
            });
            expect(
                await rewardsTokenInstance.balanceOf(ownerAddress)
            ).to.be.equalBn(ZERO_BN);
            // stake after 4 second until the 8th second
            const stakingTimestamp = startingTimestamp.add(new BN(4));
            await fastForwardTo({ timestamp: stakingTimestamp });
            await stakeAtTimestamp(
                erc20DistributionInstance,
                firstStakerAddress,
                [1],
                stakingTimestamp
            );
            const withdrawTimestamp = stakingTimestamp.add(new BN(4));
            await fastForwardTo({ timestamp: withdrawTimestamp });
            // withdraw after 4 seconds, occupying 4 seconds in total
            await withdrawAtTimestamp(
                erc20DistributionInstance,
                firstStakerAddress,
                [1],
                withdrawTimestamp
            );
            await fastForwardTo({ timestamp: endingTimestamp });

            expect(withdrawTimestamp.sub(stakingTimestamp)).to.be.equalBn(
                new BN(4)
            );
            // staker claims their reward
            const expectedReward = new BN("33333333333333333333");
            await erc20DistributionInstance.claimAll(firstStakerAddress, {
                from: firstStakerAddress,
            });
            expect(
                await rewardsTokenInstance.balanceOf(firstStakerAddress)
            ).to.be.equalBn(expectedReward);
            await erc20DistributionInstance.recoverUnassignedRewards();
            expect(
                await rewardsTokenInstance.balanceOf(ownerAddress)
            ).to.be.equalBn(new BN("66666666666666666666"));
        });

        it("should recover two thirds of the rewards when a staker stakes for a third of the distribution duration, in the end period", async () => {
            const rewardsAmount = await toWei(10, rewardsTokenInstance);
            const {
                startingTimestamp,
                endingTimestamp,
                erc20DistributionInstance,
            } = await initializeDistribution({
                from: ownerAddress,
                erc20DistributionFactoryInstance,
                stakableToken: stakableTokenInstance,
                rewardTokens: [rewardsTokenInstance],
                rewardAmounts: [rewardsAmount],
                duration: 12,
            });
            await initializeStaker({
                erc20DistributionInstance,
                stakableTokenInstance,
                stakerAddress: firstStakerAddress,
                stakableAmount: 1,
            });
            expect(
                await rewardsTokenInstance.balanceOf(ownerAddress)
            ).to.be.equalBn(ZERO_BN);
            const stakingTimestamp = startingTimestamp.add(new BN(8));
            await fastForwardTo({ timestamp: stakingTimestamp });
            await stakeAtTimestamp(
                erc20DistributionInstance,
                firstStakerAddress,
                [1],
                stakingTimestamp
            );
            await fastForwardTo({ timestamp: endingTimestamp });

            const onchainEndingTimestamp = await erc20DistributionInstance.endingTimestamp();
            expect(onchainEndingTimestamp.sub(stakingTimestamp)).to.be.equalBn(
                new BN(4)
            );
            // staker claims their reward
            const expectedReward = new BN("3333333333333333333");
            await erc20DistributionInstance.claimAll(firstStakerAddress, {
                from: firstStakerAddress,
            });
            expect(
                await rewardsTokenInstance.balanceOf(firstStakerAddress)
            ).to.be.equalBn(expectedReward);
            await erc20DistributionInstance.recoverUnassignedRewards();
            expect(
                await rewardsTokenInstance.balanceOf(ownerAddress)
            ).to.be.equalBn(new BN("6666666666666666666"));
        });

        it("should recover the unassigned rewards when a staker stakes for a certain period, withdraws, stakes again, and withdraws again", async () => {
            const rewardsAmount = await toWei(100, rewardsTokenInstance);
            const {
                startingTimestamp,
                endingTimestamp,
                erc20DistributionInstance,
            } = await initializeDistribution({
                from: ownerAddress,
                erc20DistributionFactoryInstance,
                stakableToken: stakableTokenInstance,
                rewardTokens: [rewardsTokenInstance],
                rewardAmounts: [rewardsAmount],
                duration: 12,
            });
            await initializeStaker({
                erc20DistributionInstance,
                stakableTokenInstance,
                stakerAddress: firstStakerAddress,
                stakableAmount: 1,
            });
            expect(
                await rewardsTokenInstance.balanceOf(ownerAddress)
            ).to.be.equalBn(ZERO_BN);
            const firstStakingTimestamp = startingTimestamp;
            await fastForwardTo({ timestamp: firstStakingTimestamp });
            await stakeAtTimestamp(
                erc20DistributionInstance,
                firstStakerAddress,
                [1],
                firstStakingTimestamp
            );

            const firstWithdrawTimestamp = firstStakingTimestamp.add(new BN(3));
            await fastForwardTo({ timestamp: firstWithdrawTimestamp });
            await withdrawAtTimestamp(
                erc20DistributionInstance,
                firstStakerAddress,
                [1],
                firstWithdrawTimestamp
            );

            const secondStakingTimestamp = firstWithdrawTimestamp.add(
                new BN(3)
            );
            // reapproving the stakable token before staking for a second time
            await stakableTokenInstance.approve(
                erc20DistributionInstance.address,
                1,
                { from: firstStakerAddress }
            );
            await stopMining();
            await fastForwardTo({ timestamp: secondStakingTimestamp });
            // should be able to immediately claim the first unassigned rewards from the first 3 empty seconds
            await claimAllAtTimestamp(
                erc20DistributionInstance,
                firstStakerAddress,
                firstStakerAddress,
                secondStakingTimestamp
            );
            await recoverUnassignedRewardsAtTimestamp(
                erc20DistributionInstance,
                firstStakerAddress,
                secondStakingTimestamp
            );
            await stakeAtTimestamp(
                erc20DistributionInstance,
                firstStakerAddress,
                [1],
                secondStakingTimestamp
            );
            expect(await getEvmTimestamp()).to.be.equalBn(
                secondStakingTimestamp
            );
            await startMining();
            // recoverable unassigned rewards should have been put to 0
            expect(
                await erc20DistributionInstance.recoverableUnassignedReward(
                    rewardsTokenInstance.address
                )
            ).to.be.equalBn(ZERO_BN);

            const secondWithdrawTimestamp = secondStakingTimestamp.add(
                new BN(3)
            );
            await fastForwardTo({ timestamp: secondWithdrawTimestamp });
            await withdrawAtTimestamp(
                erc20DistributionInstance,
                firstStakerAddress,
                [1],
                secondWithdrawTimestamp
            );

            await fastForwardTo({ timestamp: endingTimestamp });

            // the staker staked for 6 seconds total
            const expectedReward = await toWei("50", rewardsTokenInstance);
            // claiming for the second time
            await erc20DistributionInstance.claimAll(firstStakerAddress, {
                from: firstStakerAddress,
            });
            expect(
                await rewardsTokenInstance.balanceOf(firstStakerAddress)
            ).to.be.equalBn(expectedReward);

            // the owner should already have some recovered reward tokens from above
            const expectedRemainingReward = await toWei(
                "25",
                rewardsTokenInstance
            );
            expect(
                await rewardsTokenInstance.balanceOf(ownerAddress)
            ).to.be.equalBn(expectedRemainingReward);
            expect(
                await erc20DistributionInstance.recoverableUnassignedReward(
                    rewardsTokenInstance.address
                )
            ).to.be.equalBn(expectedRemainingReward);
            // claiming the unassigned rewards that accrued starting from the second withdraw
            await erc20DistributionInstance.recoverUnassignedRewards();
            expect(
                await erc20DistributionInstance.recoverableUnassignedReward(
                    rewardsTokenInstance.address
                )
            ).to.be.equalBn(ZERO_BN);
            expect(
                await rewardsTokenInstance.balanceOf(ownerAddress)
            ).to.be.equalBn(expectedRemainingReward.mul(new BN(2)));
        });

        it("should recover the unassigned rewards when a staker stakes for a certain period, withdraws, stakes again, withdraws again, and there's a direct transfer of rewards in the contract", async () => {
            const rewardsAmount = await toWei(100, rewardsTokenInstance);
            const {
                startingTimestamp,
                endingTimestamp,
                erc20DistributionInstance,
            } = await initializeDistribution({
                from: ownerAddress,
                erc20DistributionFactoryInstance,
                stakableToken: stakableTokenInstance,
                rewardTokens: [rewardsTokenInstance],
                rewardAmounts: [rewardsAmount],
                duration: 12,
            });
            await initializeStaker({
                erc20DistributionInstance,
                stakableTokenInstance,
                stakerAddress: firstStakerAddress,
                stakableAmount: 1,
            });
            // directly mint rewards to the contract (should be recovered at the first recover call)
            const firstMintedAmount = await toWei(10, rewardsTokenInstance);
            await rewardsTokenInstance.mint(
                erc20DistributionInstance.address,
                firstMintedAmount
            );
            expect(
                await rewardsTokenInstance.balanceOf(ownerAddress)
            ).to.be.equalBn(ZERO_BN);
            const firstStakingTimestamp = startingTimestamp;
            await fastForwardTo({ timestamp: firstStakingTimestamp });
            await stakeAtTimestamp(
                erc20DistributionInstance,
                firstStakerAddress,
                [1],
                firstStakingTimestamp
            );

            const firstWithdrawTimestamp = firstStakingTimestamp.add(new BN(3));
            await fastForwardTo({ timestamp: firstWithdrawTimestamp });
            await withdrawAtTimestamp(
                erc20DistributionInstance,
                firstStakerAddress,
                [1],
                firstWithdrawTimestamp
            );

            const secondStakingTimestamp = firstWithdrawTimestamp.add(
                new BN(3)
            );
            // reapproving the stakable token before staking for a second time
            await stakableTokenInstance.approve(
                erc20DistributionInstance.address,
                1,
                { from: firstStakerAddress }
            );
            await stopMining();
            await fastForwardTo({ timestamp: secondStakingTimestamp });
            // should be able to immediately claim the first unassigned rewards from the first 3 empty seconds
            await claimAllAtTimestamp(
                erc20DistributionInstance,
                firstStakerAddress,
                firstStakerAddress,
                secondStakingTimestamp
            );
            // should recover the first direct reward token transfer
            await recoverUnassignedRewardsAtTimestamp(
                erc20DistributionInstance,
                firstStakerAddress,
                secondStakingTimestamp
            );
            await stakeAtTimestamp(
                erc20DistributionInstance,
                firstStakerAddress,
                [1],
                secondStakingTimestamp
            );
            expect(await getEvmTimestamp()).to.be.equalBn(
                secondStakingTimestamp
            );
            await startMining();
            // recoverable unassigned rewards should have been put to 0
            expect(
                await erc20DistributionInstance.recoverableUnassignedReward(
                    rewardsTokenInstance.address
                )
            ).to.be.equalBn(ZERO_BN);

            // directly mint rewards to the contract for the second time
            // (should be recovered at the first recover call)
            const secondMintedAmount = await toWei(20, rewardsTokenInstance);
            await rewardsTokenInstance.mint(
                erc20DistributionInstance.address,
                secondMintedAmount
            );
            const secondWithdrawTimestamp = secondStakingTimestamp.add(
                new BN(3)
            );
            await fastForwardTo({ timestamp: secondWithdrawTimestamp });
            await withdrawAtTimestamp(
                erc20DistributionInstance,
                firstStakerAddress,
                [1],
                secondWithdrawTimestamp
            );

            await fastForwardTo({ timestamp: endingTimestamp });

            // the staker staked for 6 seconds total
            const expectedReward = await toWei("50", rewardsTokenInstance);
            // claiming for the second time
            await erc20DistributionInstance.claimAll(firstStakerAddress, {
                from: firstStakerAddress,
            });
            expect(
                await rewardsTokenInstance.balanceOf(firstStakerAddress)
            ).to.be.equalBn(expectedReward);

            // the owner should already have some recovered reward tokens from above
            // (also the first minted tokens)
            expect(
                await rewardsTokenInstance.balanceOf(ownerAddress)
            ).to.be.equalBn(
                firstMintedAmount.add(await toWei(25, rewardsTokenInstance))
            );
            // at this point recoverable rewards should be the minted amount sent to the contract
            // (20) plus 3 seconds when the contract did not have any staked amount  (at 100 total
            // reward tokens for a 12 seconds duration, this would be 100/12*3 = 25).
            // The total amount recoverable should be 45
            expect(
                await erc20DistributionInstance.recoverableUnassignedReward(
                    rewardsTokenInstance.address
                )
            ).to.be.equalBn(await toWei(45, rewardsTokenInstance));
            await erc20DistributionInstance.recoverUnassignedRewards();
            // claiming the unassigned rewards that accrued starting from the second withdraw
            expect(
                await erc20DistributionInstance.recoverableUnassignedReward(
                    rewardsTokenInstance.address
                )
            ).to.be.equalBn(ZERO_BN);
            expect(
                await rewardsTokenInstance.balanceOf(ownerAddress)
            ).to.be.equalBn(
                firstMintedAmount
                    .add(secondMintedAmount)
                    .add(await toWei(50, rewardsTokenInstance))
            );
        });
    }
);

```
#### test/erc20-distribution/single-tokens/staking.test.js
```javascript
const { expect } = require("chai");
const {
    initializeDistribution,
    initializeStaker,
    stakeAtTimestamp,
} = require("../../utils");
const { toWei } = require("../../utils/conversion");
const { fastForwardTo, mineBlock } = require("../../utils/network");
const { Duration } = require("luxon");
const { ZERO_BN, MAXIMUM_VARIANCE } = require("../../constants");
const BN = require("bn.js");

const ERC20StakingRewardsDistribution = artifacts.require(
    "ERC20StakingRewardsDistribution"
);
const ERC20StakingRewardsDistributionFactory = artifacts.require(
    "ERC20StakingRewardsDistributionFactory"
);
const FirstRewardERC20 = artifacts.require("FirstRewardERC20");
const ZeroDecimalsRewardERC20 = artifacts.require("ZeroDecimalsRewardERC20");
const FirstStakableERC20 = artifacts.require("FirstStakableERC20");

contract(
    "ERC20StakingRewardsDistribution - Single reward/stakable token - Staking",
    () => {
        let erc20DistributionFactoryInstance,
            rewardsTokenInstance,
            zeroDecimalsRewardTokenInstance,
            stakableTokenInstance,
            ownerAddress,
            stakerAddress;

        beforeEach(async () => {
            const accounts = await web3.eth.getAccounts();
            ownerAddress = accounts[0];
            const erc20DistributionInstance = await ERC20StakingRewardsDistribution.new(
                { from: ownerAddress }
            );
            erc20DistributionFactoryInstance = await ERC20StakingRewardsDistributionFactory.new(
                erc20DistributionInstance.address,
                { from: ownerAddress }
            );
            rewardsTokenInstance = await FirstRewardERC20.new();
            zeroDecimalsRewardTokenInstance = await ZeroDecimalsRewardERC20.new();
            stakableTokenInstance = await FirstStakableERC20.new();
            stakerAddress = accounts[1];
        });

        it("should fail when initialization has not been done", async () => {
            try {
                const erc20DistributionInstance = await ERC20StakingRewardsDistribution.new(
                    { from: ownerAddress }
                );
                await erc20DistributionInstance.stake([0]);
                throw new Error("should have failed");
            } catch (error) {
                expect(error.message).to.contain("SRD21");
            }
        });

        it("should fail when program has not yet started", async () => {
            try {
                const {
                    erc20DistributionInstance,
                } = await initializeDistribution({
                    from: ownerAddress,
                    erc20DistributionFactoryInstance,
                    stakableToken: stakableTokenInstance,
                    rewardTokens: [rewardsTokenInstance],
                    rewardAmounts: [3],
                    duration: 2,
                });
                await erc20DistributionInstance.stake([2], {
                    from: stakerAddress,
                });
                throw new Error("should have failed");
            } catch (error) {
                expect(error.message).to.contain("SRD21");
            }
        });

        it("should fail when the staker has not enough balance", async () => {
            try {
                const {
                    startingTimestamp,
                    erc20DistributionInstance,
                } = await initializeDistribution({
                    from: ownerAddress,
                    erc20DistributionFactoryInstance,
                    stakableToken: stakableTokenInstance,
                    rewardTokens: [rewardsTokenInstance],
                    rewardAmounts: [2],
                    duration: 2,
                });
                await mineBlock(startingTimestamp);
                await erc20DistributionInstance.stake([100], {
                    from: stakerAddress,
                });
                throw new Error("should have failed");
            } catch (error) {
                expect(error.message).to.contain(
                    "ERC20: transfer amount exceeds balance"
                );
            }
        });

        it("should fail when no allowance was set by the staker", async () => {
            try {
                await stakableTokenInstance.mint(stakerAddress, 1);
                const {
                    startingTimestamp,
                    erc20DistributionInstance,
                } = await initializeDistribution({
                    from: ownerAddress,
                    erc20DistributionFactoryInstance,
                    stakableToken: stakableTokenInstance,
                    rewardTokens: [rewardsTokenInstance],
                    rewardAmounts: [2],
                    duration: 2,
                });
                await mineBlock(startingTimestamp);
                await erc20DistributionInstance.stake([1], {
                    from: stakerAddress,
                });
                throw new Error("should have failed");
            } catch (error) {
                expect(error.message).to.contain(
                    "ERC20: transfer amount exceeds allowance"
                );
            }
        });

        it("should fail when not enough allowance was set by the staker", async () => {
            try {
                const {
                    startingTimestamp,
                    erc20DistributionInstance,
                } = await initializeDistribution({
                    from: ownerAddress,
                    erc20DistributionFactoryInstance,
                    stakableToken: stakableTokenInstance,
                    rewardTokens: [rewardsTokenInstance],
                    rewardAmounts: [3],
                    duration: 2,
                });
                await stakableTokenInstance.mint(stakerAddress, 1);
                await stakableTokenInstance.approve(
                    erc20DistributionInstance.address,
                    1,
                    { from: stakerAddress }
                );
                // mint additional tokens to the staker for which we
                // don't set the correct allowance
                await stakableTokenInstance.mint(stakerAddress, 1);
                await mineBlock(startingTimestamp);
                await erc20DistributionInstance.stake([2], {
                    from: stakerAddress,
                });
                throw new Error("should have failed");
            } catch (error) {
                expect(error.message).to.contain(
                    "ERC20: transfer amount exceeds allowance"
                );
            }
        });

        it("should succeed in the right conditions", async () => {
            const stakedAmount = await toWei(10, stakableTokenInstance);
            const rewardTokens = [rewardsTokenInstance];
            const {
                startingTimestamp,
                erc20DistributionInstance,
            } = await initializeDistribution({
                from: ownerAddress,
                erc20DistributionFactoryInstance,
                stakableToken: stakableTokenInstance,
                rewardTokens,
                rewardAmounts: [await toWei(1, rewardsTokenInstance)],
                duration: 2,
            });
            await initializeStaker({
                erc20DistributionInstance,
                stakableTokenInstance,
                stakerAddress: stakerAddress,
                stakableAmount: stakedAmount,
            });
            await fastForwardTo({ timestamp: startingTimestamp });
            await stakeAtTimestamp(
                erc20DistributionInstance,
                stakerAddress,
                stakedAmount,
                startingTimestamp
            );
            for (let i = 0; i < rewardTokens.length; i++) {
                expect(
                    await erc20DistributionInstance.stakedTokensOf(
                        stakerAddress
                    )
                ).to.be.equalBn(stakedAmount);
            }
            expect(
                await erc20DistributionInstance.totalStakedTokensAmount()
            ).to.be.equalBn(stakedAmount);
        });

        it("should fail when the staking cap is surpassed", async () => {
            try {
                const stakedAmount = await toWei(11, stakableTokenInstance);
                const stakingCap = await toWei(10, stakableTokenInstance);
                const {
                    startingTimestamp,
                    erc20DistributionInstance,
                } = await initializeDistribution({
                    from: ownerAddress,
                    erc20DistributionFactoryInstance,
                    stakableToken: stakableTokenInstance,
                    rewardTokens: [rewardsTokenInstance],
                    rewardAmounts: [2],
                    duration: 2,
                    stakingCap,
                });
                await initializeStaker({
                    erc20DistributionInstance,
                    stakableTokenInstance,
                    stakerAddress,
                    stakableAmount: stakedAmount,
                });
                await mineBlock(startingTimestamp);
                await erc20DistributionInstance.stake(stakedAmount, {
                    from: stakerAddress,
                });
                throw new Error("should have failed");
            } catch (error) {
                expect(error.message).to.contain("SRD10");
            }
        });

        it("should succeed when the staking cap is just hit", async () => {
            const stakedAmount = await toWei(10, stakableTokenInstance);
            const stakingCap = await toWei(10, stakableTokenInstance);
            const {
                startingTimestamp,
                erc20DistributionInstance,
            } = await initializeDistribution({
                from: ownerAddress,
                erc20DistributionFactoryInstance,
                stakableToken: stakableTokenInstance,
                rewardTokens: [rewardsTokenInstance],
                rewardAmounts: [2],
                duration: 2,
                stakingCap,
            });
            await initializeStaker({
                erc20DistributionInstance,
                stakableTokenInstance,
                stakerAddress,
                stakableAmount: stakedAmount,
            });
            await mineBlock(startingTimestamp);
            await erc20DistributionInstance.stake(stakedAmount, {
                from: stakerAddress,
            });
        });

        it("should correctly consolidate rewards when the user stakes 2 times with a low decimals token, in a lengthy campaign", async () => {
            const stakedAmount = await toWei(10, stakableTokenInstance);
            const rewardAmount = await toWei(
                10000,
                zeroDecimalsRewardTokenInstance
            );
            const duration = new BN(
                Math.floor(Duration.fromObject({ months: 1 }).toMillis() / 1000)
            );
            const {
                startingTimestamp,
                erc20DistributionInstance,
            } = await initializeDistribution({
                from: ownerAddress,
                erc20DistributionFactoryInstance,
                stakableToken: stakableTokenInstance,
                rewardTokens: [zeroDecimalsRewardTokenInstance],
                rewardAmounts: [rewardAmount],
                duration,
                stakingCap: 0,
            });
            await initializeStaker({
                erc20DistributionInstance,
                stakableTokenInstance,
                stakerAddress,
                stakableAmount: stakedAmount,
            });
            await fastForwardTo({ timestamp: startingTimestamp });
            const halfStakableAmount = stakedAmount.div(new BN(2));
            await stakeAtTimestamp(
                erc20DistributionInstance,
                stakerAddress,
                halfStakableAmount,
                startingTimestamp
            );
            const preSecondStakeReward = await erc20DistributionInstance.rewards(
                0
            );
            expect(preSecondStakeReward.perStakedToken).to.be.equalBn(ZERO_BN);
            // fast forwarding to 1/20th of the campaign
            const secondStakingTimestamp = startingTimestamp.add(
                duration.div(new BN(20))
            );
            await fastForwardTo({ timestamp: secondStakingTimestamp });
            await stakeAtTimestamp(
                erc20DistributionInstance,
                stakerAddress,
                halfStakableAmount,
                secondStakingTimestamp
            );
            const postSecondStakeRewards = await erc20DistributionInstance.rewards(
                0
            );
            // in the first stint, the staker staked alone for 1/10th of the distribution
            // calculate expected reward per staked token as if it were done onchain
            const firstStintDuration = secondStakingTimestamp.sub(
                startingTimestamp
            );
            const expectedRewardPerStakedToken = firstStintDuration
                .mul(rewardAmount)
                .mul(new BN(2).pow(new BN(112)))
                .div(halfStakableAmount.mul(duration));
            expect(postSecondStakeRewards.perStakedToken).to.be.equalBn(
                expectedRewardPerStakedToken
            );
            const onChainEarnedAmount = await erc20DistributionInstance.earnedRewardsOf(
                stakerAddress
            );
            expect(onChainEarnedAmount).to.have.length(1);
            expect(onChainEarnedAmount[0]).to.be.closeBn(
                halfStakableAmount // in order to check the consolidation of the first stint we need to use the staked amount in the first stint here, not the current onchain value, which is doubled
                    .mul(expectedRewardPerStakedToken)
                    .div(new BN(2).pow(new BN(112))),
                MAXIMUM_VARIANCE
            );
        });

        it("should fail when the user stakes but staking is disabled", async () => {
            const stakedAmount = await toWei(10, stakableTokenInstance);
            const rewardAmount = await toWei(
                10000,
                zeroDecimalsRewardTokenInstance
            );
            const duration = new BN(
                Math.floor(Duration.fromObject({ months: 1 }).toMillis() / 1000)
            );
            const {
                startingTimestamp,
                erc20DistributionInstance,
            } = await initializeDistribution({
                from: ownerAddress,
                erc20DistributionFactoryInstance,
                stakableToken: stakableTokenInstance,
                rewardTokens: [zeroDecimalsRewardTokenInstance],
                rewardAmounts: [rewardAmount],
                duration,
                stakingCap: 0,
            });
            await initializeStaker({
                erc20DistributionInstance,
                stakableTokenInstance,
                stakerAddress,
                stakableAmount: stakedAmount,
            });
            await erc20DistributionFactoryInstance.pauseStaking({
                from: ownerAddress,
            });
            await fastForwardTo({ timestamp: startingTimestamp });
            const halfStakableAmount = stakedAmount.div(new BN(2));
            try {
                await stakeAtTimestamp(
                    erc20DistributionInstance,
                    stakerAddress,
                    halfStakableAmount,
                    startingTimestamp
                );
                throw new Error("should have failed");
            } catch (error) {
                expect(error.message).to.contain("SRD25");
            }
        });
    }
);
```
#### test/erc20-distribution/single-tokens/withdrawing.test.js
```javascript
const BN = require("bn.js");
const { expect } = require("chai");
const {
    initializeDistribution,
    initializeStaker,
    withdraw,
    stakeAtTimestamp,
    withdrawAtTimestamp,
} = require("../../utils");
const { toWei } = require("../../utils/conversion");
const { fastForwardTo, getEvmTimestamp } = require("../../utils/network");

const ERC20StakingRewardsDistribution = artifacts.require(
    "ERC20StakingRewardsDistribution"
);
const ERC20StakingRewardsDistributionFactory = artifacts.require(
    "ERC20StakingRewardsDistributionFactory"
);
const FirstRewardERC20 = artifacts.require("FirstRewardERC20");
const FirstStakableERC20 = artifacts.require("FirstStakableERC20");

contract(
    "ERC20StakingRewardsDistribution - Single reward/stakable token - Withdrawing",
    () => {
        let erc20DistributionFactoryInstance,
            rewardsTokenInstance,
            stakableTokenInstance,
            ownerAddress,
            stakerAddress;

        beforeEach(async () => {
            const accounts = await web3.eth.getAccounts();
            ownerAddress = accounts[0];
            const erc20DistributionInstance = await ERC20StakingRewardsDistribution.new(
                { from: ownerAddress }
            );
            erc20DistributionFactoryInstance = await ERC20StakingRewardsDistributionFactory.new(
                erc20DistributionInstance.address,
                { from: ownerAddress }
            );
            rewardsTokenInstance = await FirstRewardERC20.new();
            stakableTokenInstance = await FirstStakableERC20.new();
            stakerAddress = accounts[1];
        });

        it("should fail when initialization has not been done", async () => {
            try {
                const erc20DistributionInstance = await ERC20StakingRewardsDistribution.new(
                    { from: ownerAddress }
                );
                await erc20DistributionInstance.withdraw(0);
                throw new Error("should have failed");
            } catch (error) {
                expect(error.message).to.contain("SRD20");
            }
        });

        it("should fail when the distribution has not yet started", async () => {
            try {
                const {
                    erc20DistributionInstance,
                } = await initializeDistribution({
                    from: ownerAddress,
                    erc20DistributionFactoryInstance,
                    stakableToken: stakableTokenInstance,
                    rewardTokens: [rewardsTokenInstance],
                    rewardAmounts: [2],
                    duration: 2,
                });
                await erc20DistributionInstance.withdraw(0);
                throw new Error("should have failed");
            } catch (error) {
                expect(error.message).to.contain("SRD20");
            }
        });

        it("should fail when the staker tries to withdraw more than what they staked", async () => {
            try {
                const {
                    startingTimestamp,
                    erc20DistributionInstance,
                } = await initializeDistribution({
                    from: ownerAddress,
                    erc20DistributionFactoryInstance,
                    stakableToken: stakableTokenInstance,
                    rewardTokens: [rewardsTokenInstance],
                    rewardAmounts: [20],
                    duration: 10,
                });
                await initializeStaker({
                    erc20DistributionInstance,
                    stakableTokenInstance,
                    stakerAddress: stakerAddress,
                    stakableAmount: 1,
                });
                await fastForwardTo({ timestamp: startingTimestamp });
                await stakeAtTimestamp(
                    erc20DistributionInstance,
                    stakerAddress,
                    1,
                    startingTimestamp
                );
                await erc20DistributionInstance.withdraw(2, {
                    from: stakerAddress,
                });
                throw new Error("should have failed");
            } catch (error) {
                expect(error.message).to.contain("SRD13");
            }
        });

        it("should succeed in the right conditions, when the distribution has not yet ended", async () => {
            const stakedAmount = await toWei(10, stakableTokenInstance);
            const {
                startingTimestamp,
                erc20DistributionInstance,
            } = await initializeDistribution({
                from: ownerAddress,
                erc20DistributionFactoryInstance,
                stakableToken: stakableTokenInstance,
                rewardTokens: [rewardsTokenInstance],
                rewardAmounts: [await toWei(1, rewardsTokenInstance)],
                duration: 10,
            });
            await initializeStaker({
                erc20DistributionInstance,
                stakableTokenInstance,
                stakerAddress: stakerAddress,
                stakableAmount: stakedAmount,
            });
            await fastForwardTo({ timestamp: startingTimestamp });
            await stakeAtTimestamp(
                erc20DistributionInstance,
                stakerAddress,
                stakedAmount,
                startingTimestamp
            );
            expect(
                await erc20DistributionInstance.stakedTokensOf(stakerAddress)
            ).to.be.equalBn(stakedAmount);
            await withdraw(
                erc20DistributionInstance,
                stakerAddress,
                stakedAmount.div(new BN(2))
            );
            expect(
                await erc20DistributionInstance.stakedTokensOf(stakerAddress)
            ).to.be.equalBn(stakedAmount.div(new BN(2)));
            expect(
                await stakableTokenInstance.balanceOf(stakerAddress)
            ).to.be.equalBn(stakedAmount.div(new BN(2)));
        });

        it("should succeed in the right conditions, when the distribution has already ended", async () => {
            const stakedAmount = await toWei(10, stakableTokenInstance);
            const {
                startingTimestamp,
                endingTimestamp,
                erc20DistributionInstance,
            } = await initializeDistribution({
                from: ownerAddress,
                erc20DistributionFactoryInstance,
                stakableToken: stakableTokenInstance,
                rewardTokens: [rewardsTokenInstance],
                rewardAmounts: [await toWei(1, rewardsTokenInstance)],
                duration: 10,
            });
            await initializeStaker({
                erc20DistributionInstance,
                stakableTokenInstance,
                stakerAddress: stakerAddress,
                stakableAmount: stakedAmount,
            });
            await fastForwardTo({ timestamp: startingTimestamp });
            await stakeAtTimestamp(
                erc20DistributionInstance,
                stakerAddress,
                stakedAmount,
                startingTimestamp
            );
            expect(
                await erc20DistributionInstance.stakedTokensOf(stakerAddress)
            ).to.be.equalBn(stakedAmount);
            await fastForwardTo({ timestamp: endingTimestamp });
            await withdrawAtTimestamp(
                erc20DistributionInstance,
                stakerAddress,
                stakedAmount.div(new BN(2)),
                endingTimestamp
            );
            expect(
                await erc20DistributionInstance.stakedTokensOf(stakerAddress)
            ).to.be.equalBn(stakedAmount.div(new BN(2)));
            expect(
                await stakableTokenInstance.balanceOf(stakerAddress)
            ).to.be.equalBn(stakedAmount.div(new BN(2)));
        });

        it("should fail when trying to withdraw from a non-ended locked distribution, right in the middle of it", async () => {
            try {
                const stakedAmount = await toWei(10, stakableTokenInstance);
                const {
                    startingTimestamp,
                    erc20DistributionInstance,
                } = await initializeDistribution({
                    from: ownerAddress,
                    erc20DistributionFactoryInstance,
                    stakableToken: stakableTokenInstance,
                    rewardTokens: [rewardsTokenInstance],
                    rewardAmounts: [await toWei(1, rewardsTokenInstance)],
                    duration: 10,
                    locked: true,
                });
                await initializeStaker({
                    erc20DistributionInstance,
                    stakableTokenInstance,
                    stakerAddress: stakerAddress,
                    stakableAmount: stakedAmount,
                });
                await fastForwardTo({ timestamp: startingTimestamp });
                await stakeAtTimestamp(
                    erc20DistributionInstance,
                    stakerAddress,
                    stakedAmount,
                    startingTimestamp
                );
                expect(
                    await erc20DistributionInstance.stakedTokensOf(
                        stakerAddress
                    )
                ).to.be.equalBn(stakedAmount);
                // fast-forward to the middle of the distribution
                const withdrawingTimestamp = startingTimestamp.add(new BN(5));
                await fastForwardTo({ timestamp: withdrawingTimestamp });
                await withdraw(
                    erc20DistributionInstance,
                    stakerAddress,
                    stakedAmount.div(new BN(2))
                );
                throw new Error("should have failed");
            } catch (error) {
                expect(error.message).to.contain("SRD12");
            }
        });

        it("should fail when trying to withdraw from a non-ended locked distribution, right at the last second of it", async () => {
            try {
                const stakedAmount = await toWei(10, stakableTokenInstance);
                const {
                    startingTimestamp,
                    endingTimestamp,
                    erc20DistributionInstance,
                } = await initializeDistribution({
                    from: ownerAddress,
                    erc20DistributionFactoryInstance,
                    stakableToken: stakableTokenInstance,
                    rewardTokens: [rewardsTokenInstance],
                    rewardAmounts: [await toWei(1, rewardsTokenInstance)],
                    duration: 10,
                    locked: true,
                });
                await initializeStaker({
                    erc20DistributionInstance,
                    stakableTokenInstance,
                    stakerAddress: stakerAddress,
                    stakableAmount: stakedAmount,
                });
                expect(
                    await stakableTokenInstance.balanceOf(stakerAddress)
                ).to.be.equalBn(stakedAmount);
                expect(await erc20DistributionInstance.locked()).to.be.true;
                await fastForwardTo({ timestamp: startingTimestamp });
                await stakeAtTimestamp(
                    erc20DistributionInstance,
                    stakerAddress,
                    stakedAmount,
                    startingTimestamp
                );
                expect(
                    await erc20DistributionInstance.stakedTokensOf(
                        stakerAddress
                    )
                ).to.be.equalBn(stakedAmount);
                // fast-forward to the middle of the distribution
                const withdrawingTimestamp = endingTimestamp;
                await fastForwardTo({
                    timestamp: withdrawingTimestamp,
                    mineBlockAfter: false,
                });
                await withdrawAtTimestamp(
                    erc20DistributionInstance,
                    stakerAddress,
                    stakedAmount.div(new BN(2)),
                    withdrawingTimestamp
                );
                expect(await getEvmTimestamp()).to.be.equalBn(
                    withdrawingTimestamp
                );
                throw new Error("should have failed");
            } catch (error) {
                expect(error.message).to.contain("SRD12");
            }
        });

        it("should succeed when withdrawing from an ended locked distribution", async () => {
            const stakedAmount = await toWei(10, stakableTokenInstance);
            const {
                startingTimestamp,
                endingTimestamp,
                erc20DistributionInstance,
            } = await initializeDistribution({
                from: ownerAddress,
                erc20DistributionFactoryInstance,
                stakableToken: stakableTokenInstance,
                rewardTokens: [rewardsTokenInstance],
                rewardAmounts: [await toWei(1, rewardsTokenInstance)],
                duration: 10,
                locked: true,
            });
            await initializeStaker({
                erc20DistributionInstance,
                stakableTokenInstance,
                stakerAddress: stakerAddress,
                stakableAmount: stakedAmount,
            });
            await fastForwardTo({ timestamp: startingTimestamp });
            await stakeAtTimestamp(
                erc20DistributionInstance,
                stakerAddress,
                stakedAmount,
                startingTimestamp
            );
            expect(
                await erc20DistributionInstance.stakedTokensOf(stakerAddress)
            ).to.be.equalBn(stakedAmount);
            // fast-forward to the middle of the distribution
            const withdrawingTimestamp = endingTimestamp.add(new BN(2));
            await fastForwardTo({ timestamp: withdrawingTimestamp });
            await withdrawAtTimestamp(
                erc20DistributionInstance,
                stakerAddress,
                stakedAmount,
                withdrawingTimestamp
            );
            expect(await getEvmTimestamp()).to.be.equalBn(withdrawingTimestamp);
            expect(
                await stakableTokenInstance.balanceOf(stakerAddress)
            ).to.be.equalBn(stakedAmount);
        });
    }
);
```
#### test/erc20-distribution-factory/multi-rewards-single-stakable/create-distribution.test.js
```javascript
const BN = require("bn.js");
const { expect } = require("chai");
const { getEvmTimestamp } = require("../../utils/network");

const ERC20StakingRewardsDistributionFactory = artifacts.require(
    "ERC20StakingRewardsDistributionFactory"
);
const ERC20StakingRewardsDistribution = artifacts.require(
    "ERC20StakingRewardsDistribution"
);
const UpgradedERC20StakingRewardsDistribution = artifacts.require(
    "UpgradedERC20StakingRewardsDistribution"
);
const FirstRewardERC20 = artifacts.require("FirstRewardERC20");
const SecondRewardERC20 = artifacts.require("SecondRewardERC20");
const FirstStakableERC20 = artifacts.require("FirstStakableERC20");

contract(
    "ERC20StakingRewardsDistributionFactory - Distribution creation",
    () => {
        let erc20DistributionFactoryInstance,
            firstRewardsTokenInstance,
            secondRewardsTokenInstance,
            stakableTokenInstance,
            ownerAddress;

        beforeEach(async () => {
            const accounts = await web3.eth.getAccounts();
            ownerAddress = accounts[1];
            const implementation = await ERC20StakingRewardsDistribution.new();
            erc20DistributionFactoryInstance = await ERC20StakingRewardsDistributionFactory.new(
                implementation.address,
                { from: ownerAddress }
            );
            firstRewardsTokenInstance = await FirstRewardERC20.new();
            secondRewardsTokenInstance = await SecondRewardERC20.new();
            stakableTokenInstance = await FirstStakableERC20.new();
        });

        it("should fail when the caller has not enough first reward token", async () => {
            try {
                // 10 seconds from now
                const startingTimestamp = (await getEvmTimestamp()).add(
                    new BN(10)
                );
                await erc20DistributionFactoryInstance.createDistribution(
                    [
                        firstRewardsTokenInstance.address,
                        secondRewardsTokenInstance.address,
                    ],
                    stakableTokenInstance.address,
                    [10, 10],
                    startingTimestamp,
                    startingTimestamp.add(new BN(10)),
                    false,
                    0
                );
                throw new Error("should have failed");
            } catch (error) {
                expect(error.message).to.contain(
                    "ERC20: transfer amount exceeds balance"
                );
            }
        });

        it("should fail when the caller has enough first reward token, but no approval was set by the owner", async () => {
            try {
                // 10 seconds from now
                const firstRewardAmount = 20;
                await firstRewardsTokenInstance.mint(
                    ownerAddress,
                    firstRewardAmount
                );
                // no allowance given
                const startingTimestamp = (await getEvmTimestamp()).add(
                    new BN(10)
                );
                await erc20DistributionFactoryInstance.createDistribution(
                    [
                        firstRewardsTokenInstance.address,
                        secondRewardsTokenInstance.address,
                    ],
                    stakableTokenInstance.address,
                    [firstRewardAmount, 10],
                    startingTimestamp,
                    startingTimestamp.add(new BN(10)),
                    false,
                    0,
                    { from: ownerAddress }
                );
                throw new Error("should have failed");
            } catch (error) {
                expect(error.message).to.contain(
                    "ERC20: transfer amount exceeds allowance"
                );
            }
        });

        it("should fail when the caller has not enough second reward token", async () => {
            try {
                // 10 seconds from now
                const firstRewardAmount = 20;
                await firstRewardsTokenInstance.mint(
                    ownerAddress,
                    firstRewardAmount
                );
                await firstRewardsTokenInstance.approve(
                    erc20DistributionFactoryInstance.address,
                    firstRewardAmount,
                    { from: ownerAddress }
                );
                const startingTimestamp = (await getEvmTimestamp()).add(
                    new BN(10)
                );
                await erc20DistributionFactoryInstance.createDistribution(
                    [
                        firstRewardsTokenInstance.address,
                        secondRewardsTokenInstance.address,
                    ],
                    stakableTokenInstance.address,
                    [firstRewardAmount, 10],
                    startingTimestamp,
                    startingTimestamp.add(new BN(10)),
                    false,
                    0
                );
                throw new Error("should have failed");
            } catch (error) {
                expect(error.message).to.contain(
                    "ERC20: transfer amount exceeds balance"
                );
            }
        });

        it("should fail when the caller has enough second reward token, but no approval was set by the owner", async () => {
            try {
                // 10 seconds from now
                const firstRewardAmount = 20;
                await firstRewardsTokenInstance.mint(
                    ownerAddress,
                    firstRewardAmount
                );
                await firstRewardsTokenInstance.approve(
                    erc20DistributionFactoryInstance.address,
                    firstRewardAmount,
                    { from: ownerAddress }
                );
                const secondRewardAmount = 40;
                await secondRewardsTokenInstance.mint(
                    ownerAddress,
                    secondRewardAmount
                );
                // no allowance given
                const startingTimestamp = (await getEvmTimestamp()).add(
                    new BN(10)
                );
                await erc20DistributionFactoryInstance.createDistribution(
                    [
                        firstRewardsTokenInstance.address,
                        secondRewardsTokenInstance.address,
                    ],
                    stakableTokenInstance.address,
                    [firstRewardAmount, secondRewardAmount],
                    startingTimestamp,
                    startingTimestamp.add(new BN(10)),
                    false,
                    0,
                    { from: ownerAddress }
                );
                throw new Error("should have failed");
            } catch (error) {
                expect(error.message).to.contain(
                    "ERC20: transfer amount exceeds allowance"
                );
            }
        });

        it("should succeed when in the right conditions", async () => {
            const firstRewardAmount = 10;
            await firstRewardsTokenInstance.mint(
                ownerAddress,
                firstRewardAmount
            );
            await firstRewardsTokenInstance.approve(
                erc20DistributionFactoryInstance.address,
                firstRewardAmount,
                { from: ownerAddress }
            );

            const secondRewardAmount = 20;
            await secondRewardsTokenInstance.mint(
                ownerAddress,
                secondRewardAmount
            );
            await secondRewardsTokenInstance.approve(
                erc20DistributionFactoryInstance.address,
                secondRewardAmount,
                { from: ownerAddress }
            );
            const rewardAmounts = [firstRewardAmount, secondRewardAmount];
            const rewardTokens = [
                firstRewardsTokenInstance.address,
                secondRewardsTokenInstance.address,
            ];
            const startingTimestamp = (await getEvmTimestamp()).add(new BN(10));
            const endingTimestamp = startingTimestamp.add(new BN(10));
            const locked = false;
            await erc20DistributionFactoryInstance.createDistribution(
                rewardTokens,
                stakableTokenInstance.address,
                rewardAmounts,
                startingTimestamp,
                endingTimestamp,
                locked,
                0,
                { from: ownerAddress }
            );
            expect(
                await erc20DistributionFactoryInstance.getDistributionsAmount()
            ).to.be.equalBn(new BN(1));
            const createdDistributionAddress = await erc20DistributionFactoryInstance.distributions(
                0
            );
            const erc20DistributionInstance = await ERC20StakingRewardsDistribution.at(
                createdDistributionAddress
            );

            const onchainRewardTokens = await erc20DistributionInstance.getRewardTokens();
            const onchainStakableToken = await erc20DistributionInstance.stakableToken();
            const onchainStartingTimestamp = await erc20DistributionInstance.startingTimestamp();
            const onchainEndingTimestamp = await erc20DistributionInstance.endingTimestamp();

            expect(onchainRewardTokens).to.have.length(rewardTokens.length);
            expect(onchainStakableToken).to.be.equal(
                stakableTokenInstance.address
            );
            for (let i = 0; i < onchainRewardTokens.length; i++) {
                const token = onchainRewardTokens[i];
                const amount = await erc20DistributionInstance.rewardAmount(
                    token
                );
                expect(amount.toNumber()).to.be.equal(rewardAmounts[i]);
            }
            expect(onchainStartingTimestamp).to.be.equalBn(startingTimestamp);
            expect(onchainEndingTimestamp).to.be.equalBn(endingTimestamp);
            expect(await erc20DistributionInstance.owner()).to.be.equal(
                ownerAddress
            );
        });

        it("should succeed when upgrading the implementation (new campaigns must use the new impl, old ones the previous one)", async () => {
            const firstRewardAmount = 20;
            await firstRewardsTokenInstance.mint(
                ownerAddress,
                firstRewardAmount
            );
            await firstRewardsTokenInstance.approve(
                erc20DistributionFactoryInstance.address,
                firstRewardAmount,
                { from: ownerAddress }
            );

            const secondRewardAmount = 40;
            await secondRewardsTokenInstance.mint(
                ownerAddress,
                secondRewardAmount
            );
            await secondRewardsTokenInstance.approve(
                erc20DistributionFactoryInstance.address,
                secondRewardAmount,
                { from: ownerAddress }
            );
            const rewardAmounts = [
                firstRewardAmount / 2,
                secondRewardAmount / 2,
            ];
            const rewardTokens = [
                firstRewardsTokenInstance.address,
                secondRewardsTokenInstance.address,
            ];
            const startingTimestamp = (await getEvmTimestamp()).add(new BN(10));
            const endingTimestamp = startingTimestamp.add(new BN(10));
            const locked = false;
            // proxy 1
            await erc20DistributionFactoryInstance.createDistribution(
                rewardTokens,
                stakableTokenInstance.address,
                rewardAmounts,
                startingTimestamp,
                endingTimestamp,
                locked,
                0,
                { from: ownerAddress }
            );

            // upgrading implementation
            const upgradedDistribution = await UpgradedERC20StakingRewardsDistribution.new();
            await erc20DistributionFactoryInstance.upgradeImplementation(
                upgradedDistribution.address,
                { from: ownerAddress }
            );

            // proxy 2
            await erc20DistributionFactoryInstance.createDistribution(
                rewardTokens,
                stakableTokenInstance.address,
                rewardAmounts,
                startingTimestamp,
                endingTimestamp,
                locked,
                0,
                { from: ownerAddress }
            );
            expect(
                await erc20DistributionFactoryInstance.getDistributionsAmount()
            ).to.be.equalBn(new BN(2));
            const proxy1Address = await erc20DistributionFactoryInstance.distributions(
                0
            );
            const proxy2Address = await erc20DistributionFactoryInstance.distributions(
                1
            );
            const distribution1Instance = await UpgradedERC20StakingRewardsDistribution.at(
                proxy1Address
            );
            const distribution2Instance = await UpgradedERC20StakingRewardsDistribution.at(
                proxy2Address
            );

            try {
                await distribution1Instance.isUpgraded();
                throw new Error("should have failed");
            } catch (error) {
                expect(error.message).to.contain("revert");
            }

            expect(await distribution2Instance.isUpgraded()).to.be.true;
        });
    }
);
```
#### test/erc20-distribution-factory/single-tokens/create-distribution.test.js
```javascript
const BN = require("bn.js");
const { expect } = require("chai");
const { getEvmTimestamp } = require("../../utils/network");

const ERC20StakingRewardsDistributionFactory = artifacts.require(
    "ERC20StakingRewardsDistributionFactory"
);
const ERC20StakingRewardsDistribution = artifacts.require(
    "ERC20StakingRewardsDistribution"
);
const FirstRewardERC20 = artifacts.require("FirstRewardERC20");
const FirstStakableERC20 = artifacts.require("FirstStakableERC20");

contract(
    "ERC20StakingRewardsDistributionFactory - Distribution creation",
    () => {
        let erc20DistributionFactoryInstance,
            erc20DistributionInstance,
            rewardsTokenInstance,
            stakableTokenInstance,
            ownerAddress;

        beforeEach(async () => {
            const accounts = await web3.eth.getAccounts();
            ownerAddress = accounts[1];
            erc20DistributionInstance = await ERC20StakingRewardsDistribution.new();
            erc20DistributionFactoryInstance = await ERC20StakingRewardsDistributionFactory.new(
                erc20DistributionInstance.address,
                { from: ownerAddress }
            );
            rewardsTokenInstance = await FirstRewardERC20.new();
            stakableTokenInstance = await FirstStakableERC20.new();
        });

        it("should fail when the caller has not enough reward token", async () => {
            try {
                // 10 seconds from now
                const startingTimestamp = (await getEvmTimestamp()).add(
                    new BN(10)
                );
                await erc20DistributionFactoryInstance.createDistribution(
                    [rewardsTokenInstance.address],
                    stakableTokenInstance.address,
                    [10],
                    startingTimestamp,
                    startingTimestamp.add(new BN(10)),
                    false,
                    0
                );
                throw new Error("should have failed");
            } catch (error) {
                expect(error.message).to.contain(
                    "ERC20: transfer amount exceeds balance"
                );
            }
        });

        it("should fail when the caller has enough reward token but no approval was given to the factory contract", async () => {
            try {
                // 10 seconds from now
                const rewardAmount = 10;
                await rewardsTokenInstance.mint(ownerAddress, rewardAmount);
                const startingTimestamp = (await getEvmTimestamp()).add(
                    new BN(10)
                );
                await erc20DistributionFactoryInstance.createDistribution(
                    [rewardsTokenInstance.address],
                    stakableTokenInstance.address,
                    [rewardAmount],
                    startingTimestamp,
                    startingTimestamp.add(new BN(10)),
                    false,
                    0,
                    { from: ownerAddress }
                );
                throw new Error("should have failed");
            } catch (error) {
                expect(error.message).to.contain(
                    "ERC20: transfer amount exceeds allowance"
                );
            }
        });

        it("should succeed when in the right conditions", async () => {
            // 10 seconds from now
            const rewardAmount = 10;
            await rewardsTokenInstance.mint(ownerAddress, rewardAmount);
            await rewardsTokenInstance.approve(
                erc20DistributionFactoryInstance.address,
                rewardAmount,
                { from: ownerAddress }
            );
            const rewardAmounts = [rewardAmount];
            const rewardTokens = [rewardsTokenInstance.address];
            const startingTimestamp = (await getEvmTimestamp()).add(new BN(10));
            const endingTimestamp = startingTimestamp.add(new BN(10));
            const locked = false;
            await erc20DistributionFactoryInstance.createDistribution(
                rewardTokens,
                stakableTokenInstance.address,
                rewardAmounts,
                startingTimestamp,
                endingTimestamp,
                locked,
                0,
                { from: ownerAddress }
            );
            expect(
                await erc20DistributionFactoryInstance.getDistributionsAmount()
            ).to.be.equalBn(new BN(1));
            const createdDistributionAddress = await erc20DistributionFactoryInstance.distributions(
                0
            );
            const erc20DistributionInstance = await ERC20StakingRewardsDistribution.at(
                createdDistributionAddress
            );

            const onchainRewardTokens = await erc20DistributionInstance.getRewardTokens();
            const onchainStartingTimestamp = await erc20DistributionInstance.startingTimestamp();
            const onchainEndingTimestamp = await erc20DistributionInstance.endingTimestamp();

            expect(onchainRewardTokens).to.have.length(rewardTokens.length);
            expect(await erc20DistributionInstance.stakableToken()).to.be.equal(
                stakableTokenInstance.address
            );
            for (let i = 0; i < onchainRewardTokens.length; i++) {
                const token = onchainRewardTokens[i];
                const amount = await erc20DistributionInstance.rewardAmount(
                    token
                );
                expect(amount.toNumber()).to.be.equal(rewardAmounts[i]);
            }
            expect(onchainStartingTimestamp).to.be.equalBn(startingTimestamp);
            expect(onchainEndingTimestamp).to.be.equalBn(endingTimestamp);
            expect(await erc20DistributionInstance.owner()).to.be.equal(
                ownerAddress
            );
        });
    }
);
```
#### test/erc20-distribution-factory/single-tokens/pause-staking.test.js
```javascript
const { expect } = require("chai");

const ERC20StakingRewardsDistributionFactory = artifacts.require(
    "ERC20StakingRewardsDistributionFactory"
);
const ERC20StakingRewardsDistribution = artifacts.require(
    "ERC20StakingRewardsDistribution"
);

contract("ERC20StakingRewardsDistributionFactory - Pause staking", () => {
    let erc20DistributionFactoryInstance,
        erc20DistributionInstance,
        ownerAddress;

    beforeEach(async () => {
        const accounts = await web3.eth.getAccounts();
        ownerAddress = accounts[1];
        erc20DistributionInstance = await ERC20StakingRewardsDistribution.new();
        erc20DistributionFactoryInstance = await ERC20StakingRewardsDistributionFactory.new(
            erc20DistributionInstance.address,
            { from: ownerAddress }
        );
    });

    it("should fail when the caller is not the owner", async () => {
        try {
            // by default the first account is used, which is not the owner
            await erc20DistributionFactoryInstance.pauseStaking();
            throw new Error("should have failed");
        } catch (error) {
            expect(error.message).to.contain(
                "Ownable: caller is not the owner"
            );
        }
    });

    it("should fail when the owner already paused the staking", async () => {
        await erc20DistributionFactoryInstance.pauseStaking({
            from: ownerAddress,
        });
        try {
            await erc20DistributionFactoryInstance.pauseStaking({
                from: ownerAddress,
            });
            throw new Error("should have failed");
        } catch (error) {
            expect(error.message).to.contain("SRF01");
        }
    });

    it("should succeed in the right conditions", async () => {
        expect(await erc20DistributionFactoryInstance.stakingPaused()).to.be
            .false;
        await erc20DistributionFactoryInstance.pauseStaking({
            from: ownerAddress,
        });
        expect(await erc20DistributionFactoryInstance.stakingPaused()).to.be
            .true;
    });
});
```
#### test/erc20-distribution-factory/single-tokens/resume-staking.test.js
```javascript
const { expect } = require("chai");

const ERC20StakingRewardsDistributionFactory = artifacts.require(
    "ERC20StakingRewardsDistributionFactory"
);
const ERC20StakingRewardsDistribution = artifacts.require(
    "ERC20StakingRewardsDistribution"
);

contract("ERC20StakingRewardsDistributionFactory - Resume staking", () => {
    let erc20DistributionFactoryInstance,
        erc20DistributionInstance,
        ownerAddress;

    beforeEach(async () => {
        const accounts = await web3.eth.getAccounts();
        ownerAddress = accounts[1];
        erc20DistributionInstance = await ERC20StakingRewardsDistribution.new();
        erc20DistributionFactoryInstance = await ERC20StakingRewardsDistributionFactory.new(
            erc20DistributionInstance.address,
            { from: ownerAddress }
        );
    });

    it("should fail when the caller is not the owner", async () => {
        try {
            // by default the first account is used, which is not the owner
            await erc20DistributionFactoryInstance.resumeStaking();
            throw new Error("should have failed");
        } catch (error) {
            expect(error.message).to.contain(
                "Ownable: caller is not the owner"
            );
        }
    });

    it("should fail when the staking is already active", async () => {
        try {
            await erc20DistributionFactoryInstance.resumeStaking({
                from: ownerAddress,
            });
            throw new Error("should have failed");
        } catch (error) {
            expect(error.message).to.contain("SRF02");
        }
    });

    it("should fail when the staking has been paused but already resumed", async () => {
        await erc20DistributionFactoryInstance.pauseStaking({
            from: ownerAddress,
        });
        await erc20DistributionFactoryInstance.resumeStaking({
            from: ownerAddress,
        });
        try {
            await erc20DistributionFactoryInstance.resumeStaking({
                from: ownerAddress,
            });
            throw new Error("should have failed");
        } catch (error) {
            expect(error.message).to.contain("SRF02");
        }
    });

    it("should succeed in the right conditions", async () => {
        expect(await erc20DistributionFactoryInstance.stakingPaused()).to.be
            .false;
        await erc20DistributionFactoryInstance.pauseStaking({
            from: ownerAddress,
        });
        expect(await erc20DistributionFactoryInstance.stakingPaused()).to.be
            .true;
        await erc20DistributionFactoryInstance.resumeStaking({
            from: ownerAddress,
        });
        expect(await erc20DistributionFactoryInstance.stakingPaused()).to.be
            .false;
    });
});
```


#### test/utils/assertion.js
```javascript
const chai = require("chai");
const { isBN } = require("bn.js");

chai.util.addMethod(
    chai.Assertion.prototype,
    "closeBn",
    function (bn2, maximumVariance) {
        const bn1 = chai.util.flag(this, "object");
        if (!isBN(bn1) || !isBN(bn2) || !isBN(maximumVariance)) {
            throw new Error("Not a BN instance");
        }
        const actualVariance = bn2.sub(bn1).abs();
        try {
            new chai.Assertion(actualVariance.lte(maximumVariance)).to.be.true;
        } catch (error) {
            throw new Error(
                `expected variance is ${maximumVariance.toString()}, but it was ${actualVariance.toString()} for numbers ${bn1.toString()} and ${bn2.toString()}`
            );
        }
    }
);

chai.util.addMethod(chai.Assertion.prototype, "equalBn", function (bn2) {
    const bn1 = chai.util.flag(this, "object");
    if (!isBN(bn1) || !isBN(bn2)) {
        throw new Error("Not a BN instance");
    }
    try {
        new chai.Assertion(bn1.eq(bn2)).to.be.true;
    } catch (error) {
        throw new Error(
            `expected ${bn1.toString()} to be the same as ${bn2.toString()}`
        );
    }
});
```
#### test/utils/conversion.js
```javascript
const BN = require("bn.js");

exports.toWei = async (amount, tokenInstance) => {
    const decimals = await tokenInstance.decimals();
    return new BN(amount).mul(new BN(10).pow(decimals));
};
```
#### test/utils/index.js
```javascript
const BN = require("bn.js");
const { ZERO_ADDRESS } = require("../constants");
const {
    getEvmTimestamp,
    stopMining,
    startMining,
    mineBlock,
} = require("./network");

const ERC20StakingRewardsDistribution = artifacts.require(
    "ERC20StakingRewardsDistribution"
);

exports.initializeStaker = async ({
    erc20DistributionInstance,
    stakableTokenInstance,
    stakerAddress,
    stakableAmount,
    setAllowance = true,
}) => {
    await stakableTokenInstance.mint(stakerAddress, stakableAmount);
    if (setAllowance) {
        await stakableTokenInstance.approve(
            erc20DistributionInstance.address,
            stakableAmount,
            { from: stakerAddress }
        );
    }
};

exports.initializeDistribution = async ({
    from,
    erc20DistributionFactoryInstance,
    stakableToken,
    rewardTokens,
    rewardAmounts,
    duration,
    startingTimestamp,
    fund = true,
    skipRewardTokensAmountsConsistenyCheck,
    locked = false,
    stakingCap = 0,
}) => {
    if (
        !skipRewardTokensAmountsConsistenyCheck &&
        rewardTokens.length !== rewardAmounts.length
    ) {
        throw new Error("reward tokens and amounts need to be the same length");
    }
    if (fund) {
        for (let i = 0; i < rewardTokens.length; i++) {
            // funds are sent directly to the distribution contract (this
            // wouldn't necessarily be needed if using the factory to
            // bootstrap distributions)
            if (rewardTokens[i].address === ZERO_ADDRESS) continue;
            await rewardTokens[i].mint(from, rewardAmounts[i]);
            await rewardTokens[i].approve(
                erc20DistributionFactoryInstance.address,
                rewardAmounts[i]
            );
        }
    }
    // if not specified, the distribution starts the next 10 second from now
    const currentEvmTimestamp = await getEvmTimestamp();
    const campaignStartingTimestamp =
        startingTimestamp && startingTimestamp.gte(currentEvmTimestamp)
            ? new BN(startingTimestamp)
            : // defaults to 10 seconds in the future
              currentEvmTimestamp.add(new BN(10));
    const campaignEndingTimestamp = campaignStartingTimestamp.add(
        new BN(duration)
    );
    await erc20DistributionFactoryInstance.createDistribution(
        rewardTokens.map((instance) => instance.address),
        stakableToken.address,
        rewardAmounts,
        campaignStartingTimestamp,
        campaignEndingTimestamp,
        locked,
        stakingCap,
        { from }
    );
    return {
        erc20DistributionInstance: await ERC20StakingRewardsDistribution.at(
            await erc20DistributionFactoryInstance.distributions(
                (await erc20DistributionFactoryInstance.getDistributionsAmount()) -
                    1
            )
        ),
        startingTimestamp: campaignStartingTimestamp,
        endingTimestamp: campaignEndingTimestamp,
    };
};

exports.initializeDistributionFromFactory = async ({
    from,
    erc20DistributionFactoryInstance,
    stakableToken,
    rewardTokens,
    rewardAmounts,
    duration,
    startingTimestamp,
    fund = true,
    skipRewardTokensAmountsConsistenyCheck,
    locked = false,
    stakingCap = 0,
}) => {
    if (
        !skipRewardTokensAmountsConsistenyCheck &&
        rewardTokens.length !== rewardAmounts.length
    ) {
        throw new Error("reward tokens and amounts need to be the same length");
    }
    if (fund) {
        for (let i = 0; i < rewardTokens.length; i++) {
            await rewardTokens[i].mint(from, rewardAmounts[i]);
            await rewardTokens[i].approve(
                erc20DistributionFactoryInstance.address,
                rewardAmounts[i]
            );
        }
    }
    // if not specified, the distribution starts the next 10 second from now
    const currentEvmTimestamp = await getEvmTimestamp();
    const campaignStartingTimestamp =
        startingTimestamp && startingTimestamp.gte(currentEvmTimestamp)
            ? new BN(startingTimestamp)
            : // defaults to 10 seconds in the future
              currentEvmTimestamp.add(new BN(10));
    const campaignEndingTimestamp = campaignStartingTimestamp.add(
        new BN(duration)
    );
    await erc20DistributionFactoryInstance.createDistribution(
        rewardTokens.map((instance) => instance.address),
        stakableToken.address,
        rewardAmounts,
        campaignStartingTimestamp,
        campaignEndingTimestamp,
        locked,
        stakingCap,
        { from }
    );
    return {
        initializedErc20DistributionInstance: ERC20StakingRewardsDistribution.at(
            await erc20DistributionFactoryInstance.distributions(
                (await erc20DistributionFactoryInstance.getDistributionsAmount()) -
                    1
            )
        ),
        startingTimestamp: campaignStartingTimestamp,
        endingTimestamp: campaignEndingTimestamp,
    };
};

exports.stake = async (
    erc20DistributionInstance,
    from,
    amount,
    waitForReceipt = true
) => {
    if (waitForReceipt) {
        await erc20DistributionInstance.stake(amount, { from });
    } else {
        // Make sure the transaction has actually been queued before returning
        return new Promise((resolve, reject) => {
            erc20DistributionInstance
                .stake(amount, { from })
                .on("transactionHash", resolve)
                .on("error", reject)
                .then(resolve)
                .catch(reject);
        });
    }
};

exports.stakeAtTimestamp = async (
    erc20DistributionInstance,
    from,
    amount,
    timestamp
) => {
    await stopMining();
    // Make sure the transaction has actually been queued before returning
    const hash = await new Promise((resolve, reject) => {
        erc20DistributionInstance
            .stake(amount, { from })
            .on("transactionHash", resolve)
            .on("error", reject)
            .then(resolve)
            .catch(reject);
    });
    await mineBlock(new BN(timestamp).toNumber());
    // By resolving the promise above when the transaction is included in the block,
    // but we need to find a way to detect reverts and error messages, to check on them in tests.
    // We can do so by getting the full transaction that was mined on-chain and "simulating"
    // it using the eth_call method (no on-chain state is changed).
    // We only do this if the transaction actually reverted on-chain after mining the block.
    // If we wouldn't perform this check, the simulation might fail because the tx changed
    // the contracts state, while if the tx reverted, we're sure to have the exact same simulation environment.
    try {
        const receipt = await web3.eth.getTransactionReceipt(hash);
        if (!receipt.status) {
            await web3.eth.call(await web3.eth.getTransaction(hash));
        }
    } finally {
        await startMining();
    }
    await startMining();
};

exports.withdraw = async (
    erc20DistributionInstance,
    from,
    amount,
    waitForReceipt = true
) => {
    if (waitForReceipt) {
        await erc20DistributionInstance.withdraw(amount, { from });
        return new BN(await web3.eth.getBlockNumber());
    } else {
        // Make sure the transaction has actually been queued before returning
        return new Promise((resolve, reject) => {
            erc20DistributionInstance
                .withdraw(amount, { from })
                .on("transactionHash", resolve)
                .on("error", reject)
                .then(resolve)
                .catch(reject);
        });
    }
};

exports.withdrawAtTimestamp = async (
    erc20DistributionInstance,
    from,
    amount,
    timestamp
) => {
    await stopMining();
    // Make sure the transaction has actually been queued before returning
    const hash = await new Promise((resolve, reject) => {
        erc20DistributionInstance
            .withdraw(amount, { from })
            .on("transactionHash", resolve)
            .on("error", reject)
            .then(resolve)
            .catch(reject);
    });
    await mineBlock(new BN(timestamp).toNumber());
    // By resolving the promise above when the transaction is included in the block,
    // but we need to find a way to detect reverts and error messages, to check on them in tests.
    // We can do so by getting the full transaction that was mined on-chain and "simulating"
    // it using the eth_call method (no on-chain state is changed).
    // We only do this if the transaction actually reverted on-chain after mining the block.
    // If we wouldn't perform this check, the simulation might fail because the tx changed
    // the contracts state, while if the tx reverted, we're sure to have the exact same simulation environment.
    try {
        const receipt = await web3.eth.getTransactionReceipt(hash);
        if (!receipt.status) {
            await web3.eth.call(await web3.eth.getTransaction(hash));
        }
    } finally {
        await startMining();
    }
};

exports.recoverUnassignedRewardsAtTimestamp = async (
    erc20DistributionInstance,
    from,
    timestamp
) => {
    await stopMining();
    // Make sure the transaction has actually been queued before returning
    const hash = await new Promise((resolve, reject) => {
        erc20DistributionInstance
            .recoverUnassignedRewards({ from })
            .on("transactionHash", resolve)
            .on("error", reject)
            .then(resolve)
            .catch(reject);
    });
    await mineBlock(new BN(timestamp).toNumber());
    // By resolving the promise above when the transaction is included in the block,
    // but we need to find a way to detect reverts and error messages, to check on them in tests.
    // We can do so by getting the full transaction that was mined on-chain and "simulating"
    // it using the eth_call method (no on-chain state is changed).
    // We only do this if the transaction actually reverted on-chain after mining the block.
    // If we wouldn't perform this check, the simulation might fail because the tx changed
    // the contracts state, while if the tx reverted, we're sure to have the exact same simulation environment.
    try {
        const receipt = await web3.eth.getTransactionReceipt(hash);
        if (!receipt.status) {
            await web3.eth.call(await web3.eth.getTransaction(hash));
        }
    } finally {
        await startMining();
    }
};

exports.claimAllAtTimestamp = async (
    erc20DistributionInstance,
    from,
    recipient,
    timestamp
) => {
    await stopMining();
    // Make sure the transaction has actually been queued before returning
    const hash = await new Promise((resolve, reject) => {
        erc20DistributionInstance
            .claimAll(recipient, { from })
            .on("transactionHash", resolve)
            .on("error", reject)
            .then(resolve)
            .catch(reject);
    });
    await mineBlock(new BN(timestamp).toNumber());
    // By resolving the promise above when the transaction is included in the block,
    // but we need to find a way to detect reverts and error messages, to check on them in tests.
    // We can do so by getting the full transaction that was mined on-chain and "simulating"
    // it using the eth_call method (no on-chain state is changed).
    // We only do this if the transaction actually reverted on-chain after mining the block.
    // If we wouldn't perform this check, the simulation might fail because the tx changed
    // the contracts state, while if the tx reverted, we're sure to have the exact same simulation environment.
    try {
        const receipt = await web3.eth.getTransactionReceipt(hash);
        if (!receipt.status) {
            await web3.eth.call(await web3.eth.getTransaction(hash));
        }
    } finally {
        await startMining();
    }
};

exports.claimPartiallyAtTimestamp = async (
    erc20DistributionInstance,
    from,
    amounts,
    recipient,
    timestamp
) => {
    await stopMining();
    // Make sure the transaction has actually been queued before returning
    const hash = await new Promise((resolve, reject) => {
        erc20DistributionInstance
            .claim(amounts, recipient, { from })
            .on("transactionHash", resolve)
            .on("error", reject)
            .then(resolve)
            .catch(reject);
    });
    await mineBlock(new BN(timestamp).toNumber());
    // By resolving the promise above when the transaction is included in the block,
    // but we need to find a way to detect reverts and error messages, to check on them in tests.
    // We can do so by getting the full transaction that was mined on-chain and "simulating"
    // it using the eth_call method (no on-chain state is changed).
    // We only do this if the transaction actually reverted on-chain after mining the block.
    // If we wouldn't perform this check, the simulation might fail because the tx changed
    // the contracts state, while if the tx reverted, we're sure to have the exact same simulation environment.
    try {
        const receipt = await web3.eth.getTransactionReceipt(hash);
        if (!receipt.status) {
            await web3.eth.call(await web3.eth.getTransaction(hash));
        }
    } finally {
        await startMining();
    }
};
```
#### test/utils/network.js
```javascript
const BN = require("bn.js");

const getEvmTimestamp = async () => {
    const { timestamp } = await web3.eth.getBlock("latest");
    return new BN(timestamp);
};
exports.getEvmTimestamp = getEvmTimestamp;

exports.stopMining = async () => {
    return new Promise((resolve, reject) => {
        web3.currentProvider.send({ method: "miner_stop" }, (error) => {
            if (error) {
                console.error("error stopping instamining", error);
                return reject(error);
            }
            return resolve();
        });
    });
};

exports.startMining = async () => {
    return new Promise((resolve, reject) => {
        web3.currentProvider.send({ method: "miner_start" }, (error) => {
            if (error) {
                console.error("error resuming instamining", error);
                return reject(error);
            }
            return resolve();
        });
    });
};

const mineBlock = async (timestamp) => {
    return new Promise((resolve, reject) => {
        web3.currentProvider.send(
            {
                id: Date.now(),
                jsonrpc: "2.0",
                method: "evm_mine",
                params: timestamp ? [new BN(timestamp).toNumber()] : [],
            },
            (error) => {
                if (error) {
                    console.error("error mining block", error);
                    return reject(error);
                }
                return resolve();
            }
        );
    });
};
exports.mineBlock = mineBlock;

exports.fastForwardTo = async ({ timestamp, mineBlockAfter = true }) => {
    const evmTimestamp = await getEvmTimestamp();
    return new Promise((resolve, reject) => {
        const secondsInterval =
            new BN(timestamp).toNumber() - evmTimestamp.toNumber();
        web3.currentProvider.send(
            {
                method: "evm_increaseTime",
                params: [secondsInterval],
            },
            (error) => {
                if (error) {
                    console.error("error mining blocks", error);
                    return reject(error);
                }
                if (mineBlockAfter) {
                    // mining a block persists the increased time on-chain. This lets us fetch the current EVM
                    // timestamp by querying the last mined block and taking its timestamp.
                    mineBlock()
                        .then(resolve)
                        .catch((error) => {
                            console.error(
                                "error mining block after fast forwarding",
                                error
                            );
                            return reject(error);
                        });
                } else {
                    resolve();
                }
            }
        );
    });
};
```

```bash
yarn test
```

#### Result must be something like this
```javascript

  Contract: ERC20StakingRewardsDistributionFactory - Distribution creation
    ✓ should fail when the caller has not enough first reward token (452ms)
    ✓ should fail when the caller has enough first reward token, but no approval was set by the owner (796ms)
    ✓ should fail when the caller has not enough second reward token (918ms)
    ✓ should fail when the caller has enough second reward token, but no approval was set by the owner (2661ms)
    ✓ should succeed when in the right conditions (31384ms)
    ✓ should succeed when upgrading the implementation (new campaigns must use the new impl, old ones the previous one) (5487ms)

  Contract: ERC20StakingRewardsDistributionFactory - Distribution creation
    ✓ should fail when the caller has not enough reward token (526ms)
    ✓ should fail when the caller has enough reward token but no approval was given to the factory contract (1143ms)
    ✓ should succeed when in the right conditions (5117ms)

  Contract: ERC20StakingRewardsDistributionFactory - Pause staking
    ✓ should fail when the caller is not the owner (845ms)
    ✓ should fail when the owner already paused the staking (1383ms)
    ✓ should succeed in the right conditions (2009ms)

  Contract: ERC20StakingRewardsDistributionFactory - Resume staking
    ✓ should fail when the caller is not the owner (2026ms)
    ✓ should fail when the staking is already active (1534ms)
    ✓ should fail when the staking has been paused but already resumed (6129ms)
    ✓ should succeed in the right conditions (98975ms)
```

#### Now let's upload the ERC20-Staking-Rewards to our github, we will use this as an important dependencies to excecute the Staking Rewards Program on Frontend

### Create Staking Distributions Contracts
create new folder
```bash
mkdir figment-staking
cd figment-staking
yarn add hardhat
npx hardhat 
```

#### Add the ERC20 Staking Rewards Contract that we made before

```bash
yarn add git://github.com/my_github_name/ERC20-Staking-Rewards.git
```

#### Add Token Registry
```bash
yarn add git://github.com/Agin-DropDisco/dexswap-registry.git
```

#### Add Core Core Smart Contracts that we made in Part I
```bash
yarn add git://github.com/my_github_name/dexswap-core.git
```

#### Inside of figment-staking folder create new folder & file
**Folder Structure**
```
figment-staking
├── contracts
│   |── interfaces.js
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
├── test
|      └── default-reward-tokens-validator
|      |     └── index.js
|      └── default-stakable-tokens-validator
|      |     └── index.js
|      └── factory
|      |     └── index.js
|      └── utils
|            └── assertion.js
|            └── index.js
├── .env
├── .eslintrc
├── .solcover.js
├── commitlint.config.js
├── hardhat.config.js
├── package.json
└────────────────────────
```

#### IDEXswapERC20StakingRewardsDistributionFactory.sol
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
#### IDEXTokenRegistry.sol
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
#### IStakableTokenValidator.sol
```javascript
// SPDX-License-Identifier: GPL-3.0

pragma solidity >=0.8.0;

interface IStakableTokenValidator {
    function validateToken(address _token) external view;
}
```
#### IRewardTokensValidator.sol
```javascript
// SPDX-License-Identifier: GPL-3.0

pragma solidity >=0.8.0;

interface IRewardTokensValidator {
    function validateTokens(address[] calldata _tokens) external view;
}
```
#### TestDependencies.4.sol
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
#### TestDependencies.sol
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
#### DeXsRewardTokensValidator.sol
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
#### DeXsStakableTokenValidator.sol
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
#### DexSwapERC20StakingRewardsDistributionFactory.sol
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


