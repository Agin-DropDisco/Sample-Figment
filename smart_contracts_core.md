

## Part I

### Create Core Smart Contracts 
**Workflow**
<img src="https://gateway.pinata.cloud/ipfs/QmdcGRdqziauNSHTsxQccLzqT4px7zY2MWBwquVpvSHkhk" align="center">

### In this tutorial, you will learn :
- How to write, deploy and interact with smart contract on Polygon.
- How to verify a smart contract.

The Smart contract itself is fork of the [Uniswapv2 core smart contracts v1.0.0.](https://github.com/Uniswap/uniswap-v2-core/tree/v1.0.0) with some necesary modification for DexSwap Dapp need, including the following:
1. DexSwapDeployer
2. DexSwapFeeReceiver
3. DexSwapFeeSetter
4. DexSwapERC20 [ optional ] // *depend what network you want to use*

This tutorial is aimed at someone who has an experience with [Solidity](https://soliditylang.org/) before and wants to get a more knowledge. Hopefully this tutorial will satisfy your curiosity how to build DEFI Dapp on Polygon.

## Installation {#installation}

The installation requires Yarn 0.25 or higher (`npm install yarn --global`). It is as simple as running:

We wouldn't teach you more about Npm workspace or Yarn, but hope this will help someone who want to get started from scratch.
```bash
mkdir dexswap-core // your folder name
cd dexswap-core
truffle init
yarn init
yarn add @openzeppelin-contracts@2.5.1 && yarn add dotenv && yarn add mocha && yarn add ethereum-waffle &&  yarn add ethereumjs-util && yarn add ethers && yarn add truffle-flattener && yarn add truffle-hdwallet-provider && yarn add ts-node
```
### Create new folder inside of contract folder

```bash
cd contracts
mkdir interfaces && mkdir libraries
```
### File Structure
```
dexswap-core
|
+-- contracts
|   |
|   +-- interfaces
|   |    |
|   |    +-- IDexSwapCallee.sol
|   |    |
|   |    +-- IDexSwapERC20.sol
|   |    |
|   |    +-- IDexSwapFactory.sol
|   |    |
|   |    +-- IDexSwapPair.sol
|   |    |
|   |    +-- IERC20.sol
|   |    |
|   |    +-- IWETH.sol
|   |
|   +-- libraries
|   |    |
|   |    +-- Math.sol
|   |    |
|   |    +-- SafeMath.sol
|   |    |
|   |    +-- TransferHelper.sol
|   |    |
|   |    +-- UQ112x112.sol
|   |
|   +-- DexSwapDeployer.sol
|   |
|   +-- DexSwapERC20.sol
|   |
|   +-- DexSwapFactory.sol
|   |
|   +-- DexSwapFeeReceiver.sol
|   |
|   +-- DexSwapFeeSetter.sol
|   |
|   +-- DexSwapPair.sol
|   |
|   +-- Migrations.sol
|   |
|   +-- WETH.sol
|   |
|   |
+-- migrations
|   |    |
|   |    +-- 1_migrations.js
|   |    |
|   |    +-- 2_deploy.js
|   |
+-- scripts
|   |    |
|   |    +-- flattener.sh
|   |
+-- .env     
|      
+-- .mocharc.json 
|      
+-- .waffle.json   
|      
+--  package.json  
|      
+-- .truffle-config.js.json       
|
+-- ...
```

### Create New Solidity 
> see File Structure to get better understand how to create new folder & file

#### contracts/interfaces/IDexSwapCallee.sol
```javascript
pragma solidity >=0.5.0;

interface IDexSwapCallee {
    function DexSwapCall(address sender, uint amount0, uint amount1, bytes calldata data) external;
}
```
#### contracts/interfaces/IDexSwapERC20.sol
```javascript
pragma solidity >=0.5.0;

interface IDexSwapERC20 {
    event Approval(address indexed owner, address indexed spender, uint value);
    event Transfer(address indexed from, address indexed to, uint value);

    function name() external pure returns (string memory);
    function symbol() external pure returns (string memory);
    function decimals() external pure returns (uint8);
    function totalSupply() external view returns (uint);
    function balanceOf(address owner) external view returns (uint);
    function allowance(address owner, address spender) external view returns (uint);

    function approve(address spender, uint value) external returns (bool);
    function transfer(address to, uint value) external returns (bool);
    function transferFrom(address from, address to, uint value) external returns (bool);

    function DOMAIN_SEPARATOR() external view returns (bytes32);
    function PERMIT_TYPEHASH() external pure returns (bytes32);
    function nonces(address owner) external view returns (uint);

    function permit(address owner, address spender, uint value, uint deadline, uint8 v, bytes32 r, bytes32 s) external;
}
```
#### contracts/interfaces/IDexSwapFactory.sol
```javascript
pragma solidity >=0.5.0;

interface IDexSwapFactory {
    event PairCreated(address indexed token0, address indexed token1, address pair, uint);

    function INIT_CODE_PAIR_HASH() external pure returns (bytes32);
    function feeTo() external view returns (address);
    function protocolFeeDenominator() external view returns (uint8);
    function feeToSetter() external view returns (address);

    function getPair(address tokenA, address tokenB) external view returns (address pair);
    function allPairs(uint) external view returns (address pair);
    function allPairsLength() external view returns (uint);

    function createPair(address tokenA, address tokenB) external returns (address pair);

    function setFeeTo(address) external;
    function setFeeToSetter(address) external;
    function setProtocolFee(uint8 _protocolFee) external;
    function setSwapFee(address pair, uint32 swapFee) external;
}
```
#### contracts/interfaces/IDexSwapPair.sol
```javascript
pragma solidity >=0.5.0;

interface IDexSwapPair {
    event Approval(address indexed owner, address indexed spender, uint value);
    event Transfer(address indexed from, address indexed to, uint value);

    function name() external pure returns (string memory);
    function symbol() external pure returns (string memory);
    function decimals() external pure returns (uint8);
    function totalSupply() external view returns (uint);
    function balanceOf(address owner) external view returns (uint);
    function allowance(address owner, address spender) external view returns (uint);

    function approve(address spender, uint value) external returns (bool);
    function transfer(address to, uint value) external returns (bool);
    function transferFrom(address from, address to, uint value) external returns (bool);

    function DOMAIN_SEPARATOR() external view returns (bytes32);
    function PERMIT_TYPEHASH() external pure returns (bytes32);
    function nonces(address owner) external view returns (uint);

    function permit(address owner, address spender, uint value, uint deadline, uint8 v, bytes32 r, bytes32 s) external;

    event Mint(address indexed sender, uint amount0, uint amount1);
    event Burn(address indexed sender, uint amount0, uint amount1, address indexed to);
    event Swap(
        address indexed sender,
        uint amount0In,
        uint amount1In,
        uint amount0Out,
        uint amount1Out,
        address indexed to
    );
    event Sync(uint112 reserve0, uint112 reserve1);

    function MINIMUM_LIQUIDITY() external pure returns (uint);
    function factory() external view returns (address);
    function token0() external view returns (address);
    function token1() external view returns (address);
    function getReserves() external view returns (uint112 reserve0, uint112 reserve1, uint32 blockTimestampLast);
    function price0CumulativeLast() external view returns (uint);
    function price1CumulativeLast() external view returns (uint);
    function kLast() external view returns (uint);
    function swapFee() external view returns (uint32);

    function mint(address to) external returns (uint liquidity);
    function burn(address to) external returns (uint amount0, uint amount1);
    function swap(uint amount0Out, uint amount1Out, address to, bytes calldata data) external;
    function skim(address to) external;
    function sync() external;

    function initialize(address, address) external;
    function setSwapFee(uint32) external;
}
```
#### contracts/interfaces/IERC20.sol
```javascript
pragma solidity >=0.5.0;

interface IERC20 {
    event Approval(address indexed owner, address indexed spender, uint value);
    event Transfer(address indexed from, address indexed to, uint value);

    function name() external view returns (string memory);
    function symbol() external view returns (string memory);
    function decimals() external view returns (uint8);
    function totalSupply() external view returns (uint);
    function balanceOf(address owner) external view returns (uint);
    function allowance(address owner, address spender) external view returns (uint);

    function approve(address spender, uint value) external returns (bool);
    function transfer(address to, uint value) external returns (bool);
    function transferFrom(address from, address to, uint value) external returns (bool);
}
```
#### contracts/interfaces/IWETH.sol
```javascript
pragma solidity >=0.5.0;

interface IWETH {
    function deposit() external payable;
    function transfer(address to, uint value) external returns (bool);
    function withdraw(uint) external;
    function balanceOf(address owner) external view returns (uint);
}
```
#### contracts/libraries/Math.sol
```javascript
pragma solidity =0.5.16;

// a library for performing various math operations

library Math {
    function min(uint x, uint y) internal pure returns (uint z) {
        z = x < y ? x : y;
    }

    // babylonian method (https://en.wikipedia.org/wiki/Methods_of_computing_square_roots#Babylonian_method)
    function sqrt(uint y) internal pure returns (uint z) {
        if (y > 3) {
            z = y;
            uint x = y / 2 + 1;
            while (x < z) {
                z = x;
                x = (y / x + x) / 2;
            }
        } else if (y != 0) {
            z = 1;
        }
    }
}
```
#### contracts/libraries/SafeMath.sol
```javascript
pragma solidity =0.5.16;

// a library for performing overflow-safe math, courtesy of DappHub (https://github.com/dapphub/ds-math)

library SafeMath {
    function add(uint x, uint y) internal pure returns (uint z) {
        require((z = x + y) >= x, 'ds-math-add-overflow');
    }

    function sub(uint x, uint y) internal pure returns (uint z) {
        require((z = x - y) <= x, 'ds-math-sub-underflow');
    }

    function mul(uint x, uint y) internal pure returns (uint z) {
        require(y == 0 || (z = x * y) / y == x, 'ds-math-mul-overflow');
    }
}
```
#### contracts/libraries/TransferHelper.sol
```javascript
pragma solidity =0.5.16;

// helper methods for interacting with ERC20 tokens and sending ETH that do not consistently return true/false
library TransferHelper {
    function safeApprove(address token, address to, uint value) internal {
        // bytes4(keccak256(bytes('approve(address,uint256)')));
        (bool success, bytes memory data) = token.call(abi.encodeWithSelector(0x095ea7b3, to, value));
        require(success && (data.length == 0 || abi.decode(data, (bool))), 'TransferHelper: APPROVE_FAILED');
    }

    function safeTransfer(address token, address to, uint value) internal {
        // bytes4(keccak256(bytes('transfer(address,uint256)')));
        (bool success, bytes memory data) = token.call(abi.encodeWithSelector(0xa9059cbb, to, value));
        require(success && (data.length == 0 || abi.decode(data, (bool))), 'TransferHelper: TRANSFER_FAILED');
    }

    function safeTransferFrom(address token, address from, address to, uint value) internal {
        // bytes4(keccak256(bytes('transferFrom(address,address,uint256)')));
        (bool success, bytes memory data) = token.call(abi.encodeWithSelector(0x23b872dd, from, to, value));
        require(success && (data.length == 0 || abi.decode(data, (bool))), 'TransferHelper: TRANSFER_FROM_FAILED');
    }

    function safeTransferETH(address to, uint value) internal {
        (bool success,) = to.call.value(value)(new bytes(0));
        require(success, 'TransferHelper: ETH_TRANSFER_FAILED');
    }
}
```
#### contracts/libraries/UQ112x112.sol
```javascript
pragma solidity =0.5.16;

// a library for handling binary fixed point numbers (https://en.wikipedia.org/wiki/Q_(number_format))

// range: [0, 2**112 - 1]
// resolution: 1 / 2**112

library UQ112x112 {
    uint224 constant Q112 = 2**112;

    // encode a uint112 as a UQ112x112
    function encode(uint112 y) internal pure returns (uint224 z) {
        z = uint224(y) * Q112; // never overflows
    }

    // divide a UQ112x112 by a uint112, returning a UQ112x112
    function uqdiv(uint224 x, uint112 y) internal pure returns (uint224 z) {
        z = x / uint224(y);
    }
}
```
#### contracts/DexSwapERC20.sol
```javascript
// SPDX-License-Identifier: GPL-3.0
pragma solidity =0.5.16;

import './interfaces/IDexSwapERC20.sol';
import './libraries/SafeMath.sol';

contract DexSwapERC20 is IDexSwapERC20 {
    using SafeMath for uint;

    string public constant name = 'DexSwap';
    string public constant symbol = 'DEXS';
    uint8 public constant decimals = 18;
    uint  public totalSupply;
    mapping(address => uint) public balanceOf;
    mapping(address => mapping(address => uint)) public allowance;

    bytes32 public DOMAIN_SEPARATOR;
    // keccak256("Permit(address owner,address spender,uint256 value,uint256 nonce,uint256 deadline)");
    bytes32 public constant PERMIT_TYPEHASH = 0x6e71edae12b1b97f4d1f60370fef10105fa2faae0126114a169c64845d6126c9;
    mapping(address => uint) public nonces;

    event Approval(address indexed owner, address indexed spender, uint value);
    event Transfer(address indexed from, address indexed to, uint value);

    constructor() public {
        uint chainId;
        assembly {
            chainId := chainid
        }
        DOMAIN_SEPARATOR = keccak256(
            abi.encode(
                keccak256('EIP712Domain(string name,string version,uint256 chainId,address verifyingContract)'),
                keccak256(bytes(name)),
                keccak256(bytes('1')),
                chainId,
                address(this)
            )
        );
    }

    function _mint(address to, uint value) internal {
        totalSupply = totalSupply.add(value);
        balanceOf[to] = balanceOf[to].add(value);
        emit Transfer(address(0), to, value);
    }

    function _burn(address from, uint value) internal {
        balanceOf[from] = balanceOf[from].sub(value);
        totalSupply = totalSupply.sub(value);
        emit Transfer(from, address(0), value);
    }

    function _approve(address owner, address spender, uint value) private {
        allowance[owner][spender] = value;
        emit Approval(owner, spender, value);
    }

    function _transfer(address from, address to, uint value) private {
        balanceOf[from] = balanceOf[from].sub(value);
        balanceOf[to] = balanceOf[to].add(value);
        emit Transfer(from, to, value);
    }

    function approve(address spender, uint value) external returns (bool) {
        _approve(msg.sender, spender, value);
        return true;
    }

    function transfer(address to, uint value) external returns (bool) {
        _transfer(msg.sender, to, value);
        return true;
    }

    function transferFrom(address from, address to, uint value) external returns (bool) {
        if (allowance[from][msg.sender] != uint(-1)) {
            allowance[from][msg.sender] = allowance[from][msg.sender].sub(value);
        }
        _transfer(from, to, value);
        return true;
    }

    function permit(address owner, address spender, uint value, uint deadline, uint8 v, bytes32 r, bytes32 s) external {
        require(deadline >= block.timestamp, 'DexSwapERC20: EXPIRED');
        bytes32 digest = keccak256(
            abi.encodePacked(
                '\x19\x01',
                DOMAIN_SEPARATOR,
                keccak256(abi.encode(PERMIT_TYPEHASH, owner, spender, value, nonces[owner]++, deadline))
            )
        );
        address recoveredAddress = ecrecover(digest, v, r, s);
        require(recoveredAddress != address(0) && recoveredAddress == owner, 'DexSwapERC20: INVALID_SIGNATURE');
        _approve(owner, spender, value);
    }
}
```
#### contracts/DexSwapPair.sol
```javascript
// SPDX-License-Identifier: GPL-3.0
pragma solidity =0.5.16;

import './interfaces/IDexSwapPair.sol';
import './DexSwapERC20.sol';
import './libraries/Math.sol';
import './libraries/UQ112x112.sol';
import './interfaces/IERC20.sol';
import './interfaces/IDexSwapFactory.sol';
import './interfaces/IDexSwapCallee.sol';

contract DexSwapPair is IDexSwapPair, DexSwapERC20 {
    using SafeMath  for uint;
    using UQ112x112 for uint224;

    uint public constant MINIMUM_LIQUIDITY = 10**3;
    bytes4 private constant SELECTOR = bytes4(keccak256(bytes('transfer(address,uint256)')));

    address public factory;
    address public token0;
    address public token1;

    uint112 private reserve0;           // uses single storage slot, accessible via getReserves
    uint112 private reserve1;           // uses single storage slot, accessible via getReserves
    uint32  private blockTimestampLast; // uses single storage slot, accessible via getReserves

    uint public price0CumulativeLast;
    uint public price1CumulativeLast;
    uint public kLast; // reserve0 * reserve1, as of immediately after the most recent liquidity event
    uint32 public swapFee = 25; // uses 0.25% fee as default
    
    uint private unlocked = 1;
    modifier lock() {
        require(unlocked == 1, 'DexSwapPair: LOCKED');
        unlocked = 0;
        _;
        unlocked = 1;
    }

    function getReserves() public view returns (uint112 _reserve0, uint112 _reserve1, uint32 _blockTimestampLast) {
        _reserve0 = reserve0;
        _reserve1 = reserve1;
        _blockTimestampLast = blockTimestampLast;
    }

    function _safeTransfer(address token, address to, uint value) private {
        (bool success, bytes memory data) = token.call(abi.encodeWithSelector(SELECTOR, to, value));
        require(success && (data.length == 0 || abi.decode(data, (bool))), 'DexSwapPair: TRANSFER_FAILED');
    }

    event Mint(address indexed sender, uint amount0, uint amount1);
    event Burn(address indexed sender, uint amount0, uint amount1, address indexed to);
    event Swap(
        address indexed sender,
        uint amount0In,
        uint amount1In,
        uint amount0Out,
        uint amount1Out,
        address indexed to
    );
    event Sync(uint112 reserve0, uint112 reserve1);

    constructor() public {
        factory = msg.sender;
    }

    // called once by the factory at time of deployment
    function initialize(address _token0, address _token1) external {
        require(msg.sender == factory, 'DexSwapPair: FORBIDDEN'); // sufficient check
        token0 = _token0;
        token1 = _token1;
    }
    
    // called by the factory to set the swapFee
    function setSwapFee(uint32 _swapFee) external {
        require(msg.sender == factory, 'DexSwapPair: FORBIDDEN'); // sufficient check
        require(_swapFee <= 1000, 'DexSwapPair: FORBIDDEN_FEE'); // fee percentage check
        swapFee = _swapFee;
    }

    // update reserves and, on the first call per block, price accumulators
    function _update(uint balance0, uint balance1, uint112 _reserve0, uint112 _reserve1) private {
        require(balance0 <= uint112(-1) && balance1 <= uint112(-1), 'DexSwapPair: OVERFLOW');
        uint32 blockTimestamp = uint32(block.timestamp % 2**32);
        uint32 timeElapsed = blockTimestamp - blockTimestampLast; // overflow is desired
        if (timeElapsed > 0 && _reserve0 != 0 && _reserve1 != 0) {
            // * never overflows, and + overflow is desired
            price0CumulativeLast += uint(UQ112x112.encode(_reserve1).uqdiv(_reserve0)) * timeElapsed;
            price1CumulativeLast += uint(UQ112x112.encode(_reserve0).uqdiv(_reserve1)) * timeElapsed;
        }
        reserve0 = uint112(balance0);
        reserve1 = uint112(balance1);
        blockTimestampLast = blockTimestamp;
        emit Sync(reserve0, reserve1);
    }

    // if fee is on, mint liquidity equivalent to 1/ (protocolFeeDenominator + ~1) share of the growth in sqrt(k)
    function _mintFee(uint112 _reserve0, uint112 _reserve1) private returns (bool feeOn) {
        address feeTo = IDexSwapFactory(factory).feeTo();
        uint8 protocolFeeDenominator = IDexSwapFactory(factory).protocolFeeDenominator();
        feeOn = feeTo != address(0);
        uint _kLast = kLast; // gas savings
        if (feeOn) {
            if (_kLast != 0) {
                uint rootK = Math.sqrt(uint(_reserve0).mul(_reserve1));
                uint rootKLast = Math.sqrt(_kLast);
                if (rootK > rootKLast) {
                    uint numerator = totalSupply.mul(rootK.sub(rootKLast));
                    uint denominator = rootK.mul(protocolFeeDenominator).add(rootKLast);
                    uint liquidity = numerator / denominator;
                    if (liquidity > 0) _mint(feeTo, liquidity);
                }
            }
        } else if (_kLast != 0) {
            kLast = 0;
        }
    }

    // this low-level function should be called from a contract which performs important safety checks
    function mint(address to) external lock returns (uint liquidity) {
        (uint112 _reserve0, uint112 _reserve1,) = getReserves(); // gas savings
        uint balance0 = IERC20(token0).balanceOf(address(this));
        uint balance1 = IERC20(token1).balanceOf(address(this));
        uint amount0 = balance0.sub(_reserve0);
        uint amount1 = balance1.sub(_reserve1);

        bool feeOn = _mintFee(_reserve0, _reserve1);
        uint _totalSupply = totalSupply; // gas savings, must be defined here since totalSupply can update in _mintFee
        if (_totalSupply == 0) {
            liquidity = Math.sqrt(amount0.mul(amount1)).sub(MINIMUM_LIQUIDITY);
           _mint(address(0), MINIMUM_LIQUIDITY); // permanently lock the first MINIMUM_LIQUIDITY tokens
        } else {
            liquidity = Math.min(amount0.mul(_totalSupply) / _reserve0, amount1.mul(_totalSupply) / _reserve1);
        }
        require(liquidity > 0, 'DexSwapPair: INSUFFICIENT_LIQUIDITY_MINTED');
        _mint(to, liquidity);

        _update(balance0, balance1, _reserve0, _reserve1);
        if (feeOn) kLast = uint(reserve0).mul(reserve1); // reserve0 and reserve1 are up-to-date
        emit Mint(msg.sender, amount0, amount1);
    }

    // this low-level function should be called from a contract which performs important safety checks
    function burn(address to) external lock returns (uint amount0, uint amount1) {
        (uint112 _reserve0, uint112 _reserve1,) = getReserves(); // gas savings
        address _token0 = token0;                                // gas savings
        address _token1 = token1;                                // gas savings
        uint balance0 = IERC20(_token0).balanceOf(address(this));
        uint balance1 = IERC20(_token1).balanceOf(address(this));
        uint liquidity = balanceOf[address(this)];

        bool feeOn = _mintFee(_reserve0, _reserve1);
        uint _totalSupply = totalSupply; // gas savings, must be defined here since totalSupply can update in _mintFee
        amount0 = liquidity.mul(balance0) / _totalSupply; // using balances ensures pro-rata distribution
        amount1 = liquidity.mul(balance1) / _totalSupply; // using balances ensures pro-rata distribution
        require(amount0 > 0 && amount1 > 0, 'DexSwapPair: INSUFFICIENT_LIQUIDITY_BURNED');
        _burn(address(this), liquidity);
        _safeTransfer(_token0, to, amount0);
        _safeTransfer(_token1, to, amount1);
        balance0 = IERC20(_token0).balanceOf(address(this));
        balance1 = IERC20(_token1).balanceOf(address(this));

        _update(balance0, balance1, _reserve0, _reserve1);
        if (feeOn) kLast = uint(reserve0).mul(reserve1); // reserve0 and reserve1 are up-to-date
        emit Burn(msg.sender, amount0, amount1, to);
    }

    // this low-level function should be called from a contract which performs important safety checks
    function swap(uint amount0Out, uint amount1Out, address to, bytes calldata data) external lock {
        require(amount0Out > 0 || amount1Out > 0, 'DexSwapPair: INSUFFICIENT_OUTPUT_AMOUNT');
        (uint112 _reserve0, uint112 _reserve1,) = getReserves(); // gas savings
        require(amount0Out < _reserve0 && amount1Out < _reserve1, 'DexSwapPair: INSUFFICIENT_LIQUIDITY');

        uint balance0;
        uint balance1;
        { // scope for _token{0,1}, avoids stack too deep errors
        address _token0 = token0;
        address _token1 = token1;
        require(to != _token0 && to != _token1, 'DexSwapPair: INVALID_TO');
        if (amount0Out > 0) _safeTransfer(_token0, to, amount0Out); // optimistically transfer tokens
        if (amount1Out > 0) _safeTransfer(_token1, to, amount1Out); // optimistically transfer tokens
        if (data.length > 0) IDexSwapCallee(to).DexSwapCall(msg.sender, amount0Out, amount1Out, data);
        balance0 = IERC20(_token0).balanceOf(address(this));
        balance1 = IERC20(_token1).balanceOf(address(this));
        }
        uint amount0In = balance0 > _reserve0 - amount0Out ? balance0 - (_reserve0 - amount0Out) : 0;
        uint amount1In = balance1 > _reserve1 - amount1Out ? balance1 - (_reserve1 - amount1Out) : 0;
        require(amount0In > 0 || amount1In > 0, 'DexSwapPair: INSUFFICIENT_INPUT_AMOUNT');
        { // scope for reserve{0,1}Adjusted, avoids stack too deep errors
          uint balance0Adjusted = balance0.mul(10000).sub(amount0In.mul(swapFee));
          uint balance1Adjusted = balance1.mul(10000).sub(amount1In.mul(swapFee));
          require(balance0Adjusted.mul(balance1Adjusted) >= uint(_reserve0).mul(_reserve1).mul(10000**2), 'DexSwapPair: K');
        }

        _update(balance0, balance1, _reserve0, _reserve1);
        emit Swap(msg.sender, amount0In, amount1In, amount0Out, amount1Out, to);
    }

    // force balances to match reserves
    function skim(address to) external lock {
        address _token0 = token0; // gas savings
        address _token1 = token1; // gas savings
        _safeTransfer(_token0, to, IERC20(_token0).balanceOf(address(this)).sub(reserve0));
        _safeTransfer(_token1, to, IERC20(_token1).balanceOf(address(this)).sub(reserve1));
    }

    // force reserves to match balances
    function sync() external lock {
        _update(IERC20(token0).balanceOf(address(this)), IERC20(token1).balanceOf(address(this)), reserve0, reserve1);
    }
}
```
#### contracts/DexSwapFactory.sol
- The factory design pattern is a pretty common pattern used in programming. The idea is simple, instead of creating objects directly, you have an object (the factory) that creates objects for you. In the case of Solidity, an object is a smart contract and so a factory will deploy new contracts for you. see: ****2_deploy.js**(The Factory will call the function of FeeSetter)

```javascript
// SPDX-License-Identifier: GPL-3.0
pragma solidity =0.5.16;

import './interfaces/IDexSwapFactory.sol';
import './DexSwapPair.sol';

contract DexSwapFactory is IDexSwapFactory {
    address public feeTo;
    address public feeToSetter;
    uint8 public protocolFeeDenominator = 9; 
    bytes32 public constant INIT_CODE_PAIR_HASH = keccak256(abi.encodePacked(type(DexSwapPair).creationCode));

    mapping(address => mapping(address => address)) public getPair;
    address[] public allPairs;

    event PairCreated(address indexed token0, address indexed token1, address pair, uint);

    constructor(address _feeToSetter) public {
        feeToSetter = _feeToSetter;
    }

    function allPairsLength() external view returns (uint) {
        return allPairs.length;
    }

    function createPair(address tokenA, address tokenB) external returns (address pair) {
        require(tokenA != tokenB, 'DexSwapFactory: IDENTICAL_ADDRESSES');
        (address token0, address token1) = tokenA < tokenB ? (tokenA, tokenB) : (tokenB, tokenA);
        require(token0 != address(0), 'DexSwapFactory: ZERO_ADDRESS');
        require(getPair[token0][token1] == address(0), 'DexSwapFactory: PAIR_EXISTS'); // single check is sufficient
        bytes memory bytecode = type(DexSwapPair).creationCode;
        bytes32 salt = keccak256(abi.encodePacked(token0, token1));
        assembly {
            pair := create2(0, add(bytecode, 32), mload(bytecode), salt)
        }
        IDexSwapPair(pair).initialize(token0, token1);
        getPair[token0][token1] = pair;
        getPair[token1][token0] = pair; // populate mapping in the reverse direction
        allPairs.push(pair);
        emit PairCreated(token0, token1, pair, allPairs.length);
    }

    function setFeeTo(address _feeTo) external {
        require(msg.sender == feeToSetter, 'DexSwapFactory: FORBIDDEN');
        feeTo = _feeTo;
    }

    function setFeeToSetter(address _feeToSetter) external {
        require(msg.sender == feeToSetter, 'DexSwapFactory: FORBIDDEN');
        feeToSetter = _feeToSetter;
    }
    
    function setProtocolFee(uint8 _protocolFeeDenominator) external {
        require(msg.sender == feeToSetter, 'DexSwapFactory: FORBIDDEN');
        require(_protocolFeeDenominator > 0, 'DexSwapFactory: FORBIDDEN_FEE');
        protocolFeeDenominator = _protocolFeeDenominator;
    }
    
    function setSwapFee(address _pair, uint32 _swapFee) external {
        require(msg.sender == feeToSetter, 'DexSwapFactory: FORBIDDEN');
        IDexSwapPair(_pair).setSwapFee(_swapFee);
    }
}
```
#### contracts/DexSwapFeeReceiver.sol
- Owner or Deployer should receive all the transactions fees & will be automatically transferred to the owner/deployer
- Factory can call the DexSwapFeeReceiver Function. eg: change receivers, change ownership etc.
```javascript
// SPDX-License-Identifier: GPL-3.0
pragma solidity =0.5.16;

import './interfaces/IDexSwapFactory.sol';
import './interfaces/IDexSwapPair.sol';
import './interfaces/IWETH.sol';
import './libraries/TransferHelper.sol';
import './libraries/SafeMath.sol';

contract DexSwapFeeReceiver {
    using SafeMath for uint;

    address public owner;
    IDexSwapFactory public factory;
    address public WETH;
    address public ethReceiver;
    address public fallbackReceiver;

    constructor(
        address _owner, address _factory, address _WETH, address _ethReceiver, address _fallbackReceiver
    ) public {
        owner = _owner;
        factory = IDexSwapFactory(_factory);
        WETH = _WETH;
        ethReceiver = _ethReceiver;
        fallbackReceiver = _fallbackReceiver;
    }
    
    function() external payable {}

    function transferOwnership(address newOwner) external {
        require(msg.sender == owner, 'DexSwapFeeReceiver: FORBIDDEN');
        owner = newOwner;
    }
    
    function changeReceivers(address _ethReceiver, address _fallbackReceiver) external {
        require(msg.sender == owner, 'DexSwapFeeReceiver: FORBIDDEN');
        ethReceiver = _ethReceiver;
        fallbackReceiver = _fallbackReceiver;
    }
    
    // Returns sorted token addresses, used to handle return values from pairs sorted in this order
    function sortTokens(address tokenA, address tokenB) internal pure returns (address token0, address token1) {
        require(tokenA != tokenB, 'DexSwapFeeReceiver: IDENTICAL_ADDRESSES');
        (token0, token1) = tokenA < tokenB ? (tokenA, tokenB) : (tokenB, tokenA);
        require(token0 != address(0), 'DexSwapFeeReceiver: ZERO_ADDRESS');
    }
    
    // Helper function to know if an address is a contract, extcodesize returns the size of the code of a smart
    //  contract in a specific address
    function isContract(address addr) internal view returns (bool) {
        uint size;
        assembly { size := extcodesize(addr) }
        return size > 0;
    }

    // Calculates the CREATE2 address for a pair without making any external calls
    // Taken from DexSwapLibrary, removed the factory parameter
    function pairFor(address tokenA, address tokenB) internal view returns (address pair) {
        (address token0, address token1) = sortTokens(tokenA, tokenB);
        pair = address(uint(keccak256(abi.encodePacked(
            hex'ff',
            factory,
            keccak256(abi.encodePacked(token0, token1)),
             hex'8d5cb477d33ed6bd41c4f92a58f79b1e620735c5408981f4f6aeb73fa189b571' 
        ))));
    }
    
    // Done with code form DexSwapRouter and DexSwapLibrary, removed the deadline argument
    function _swapTokensForETH(uint amountIn, address fromToken)
        internal
    {
        IDexSwapPair pairToUse = IDexSwapPair(pairFor(fromToken, WETH));
        
        (uint reserve0, uint reserve1,) = pairToUse.getReserves();
        (uint reserveIn, uint reserveOut) = fromToken < WETH ? (reserve0, reserve1) : (reserve1, reserve0);

        require(reserveIn > 0 && reserveOut > 0, 'DexSwapFeeReceiver: INSUFFICIENT_LIQUIDITY');
        uint amountInWithFee = amountIn.mul(uint(10000).sub(pairToUse.swapFee()));
        uint numerator = amountInWithFee.mul(reserveOut);
        uint denominator = reserveIn.mul(10000).add(amountInWithFee);
        uint amountOut = numerator / denominator;
        
        TransferHelper.safeTransfer(
            fromToken, address(pairToUse), amountIn
        );
        
        (uint amount0Out, uint amount1Out) = fromToken < WETH ? (uint(0), amountOut) : (amountOut, uint(0));
        
        pairToUse.swap(
            amount0Out, amount1Out, address(this), new bytes(0)
        );
        
        IWETH(WETH).withdraw(amountOut);
        TransferHelper.safeTransferETH(ethReceiver, amountOut);
    }

    // Transfer to the owner address the token converted into ETH if possible, if not just transfer the token.
    function _takeETHorToken(address token, uint amount) internal {
      if (token == WETH) {
        // If it is WETH, transfer directly to ETH receiver
        IWETH(WETH).withdraw(amount);
        TransferHelper.safeTransferETH(ethReceiver, amount);
      } else if (isContract(pairFor(token, WETH))) {
        // If it is not WETH and there is a direct path to WETH, swap and transfer WETH to ETH receiver
        _swapTokensForETH(amount, token);
      } else {
        // If it is not WETH and there is not a direct path to WETH, transfer tokens directly to fallback receiver
        TransferHelper.safeTransfer(token, fallbackReceiver, amount);
      }
    }
    
    // Take what was charged as protocol fee from the DexSwap pair liquidity
    function takeProtocolFee(IDexSwapPair[] calldata pairs) external {
        for (uint i = 0; i < pairs.length; i++) {
            address token0 = pairs[i].token0();
            address token1 = pairs[i].token1();
            pairs[i].transfer(address(pairs[i]), pairs[i].balanceOf(address(this)));
            (uint amount0, uint amount1) = pairs[i].burn(address(this));
            if (amount0 > 0)
                _takeETHorToken(token0, amount0);
            if (amount1 > 0)
                _takeETHorToken(token1, amount1);
        }
    }

}
```
#### contracts/DexSwapFeeSetter.sol
```javascript
// SPDX-License-Identifier: GPL-3.0
pragma solidity =0.5.16;

import './interfaces/IDexSwapFactory.sol';

contract DexSwapFeeSetter {
    address public owner;
    mapping(address => address) public pairOwners;
    IDexSwapFactory public factory;
  
    constructor(address _owner, address _factory) public {
        owner = _owner;
        factory = IDexSwapFactory(_factory);
    }

    function transferOwnership(address newOwner) external {
        require(msg.sender == owner, 'DexSwapFeeSetter: FORBIDDEN');
        owner = newOwner;
    }
    
    function transferPairOwnership(address pair, address newOwner) external {
        require(msg.sender == owner, 'DexSwapFeeSetter: FORBIDDEN');
        pairOwners[pair] = newOwner;
    }

    function setFeeTo(address feeTo) external {
        require(msg.sender == owner, 'DexSwapFeeSetter: FORBIDDEN');
        factory.setFeeTo(feeTo);
    }

    function setFeeToSetter(address feeToSetter) external {
        require(msg.sender == owner, 'DexSwapFeeSetter: FORBIDDEN');
        factory.setFeeToSetter(feeToSetter);
    }
    
    function setProtocolFee(uint8 protocolFeeDenominator) external {
        require(msg.sender == owner, 'DexSwapFeeSetter: FORBIDDEN');
        factory.setProtocolFee(protocolFeeDenominator);
    }
    
    function setSwapFee(address pair, uint32 swapFee) external {
        require((msg.sender == owner) || ((msg.sender == pairOwners[pair])), 'DexSwapFeeSetter: FORBIDDEN');
        factory.setSwapFee(pair, swapFee);
    }
}
```
#### contracts/DexSwapDeployer.sol
```javascript
// SPDX-License-Identifier: GPL-3.0
pragma solidity =0.5.16;

import './DexSwapFactory.sol';
import './interfaces/IDexSwapPair.sol';
import './DexSwapFeeSetter.sol';
import './DexSwapFeeReceiver.sol';

contract DexSwapDeployer {
    address payable public protocolFeeReceiver;
    address payable public owner;
    address public WETH;
    uint8 public state = 0;
    struct TokenPair {
        address tokenA;
        address tokenB;
        uint32 swapFee;
    }
    
    TokenPair[] public initialTokenPairs;

    event FeeReceiverDeployed(address feeReceiver);    
    event FeeSetterDeployed(address feeSetter);
    event PairFactoryDeployed(address factory);
    event PairDeployed(address pair);
        
    // Step 1: Create the deployer contract with all the needed information for deployment.
    constructor(
        address payable _protocolFeeReceiver, // we can use with sender accounts//accounts[0]
        address payable _owner, // we can use with sender accounts//accounts[0]
        address _WETH, // WETH on matic || eg Matic Mainnet WETH = 0x7ceB23fD6bC0adD59E62ac25578270cFf1b9f619
        address[] memory tokensA, // our token eg: xDEXSBASE Token Address
        address[] memory tokensB, // use WETH on Matic
        uint32[] memory swapFees // use 25 as a default
    ) public {
        owner = _owner;
        WETH = _WETH;
        protocolFeeReceiver = _protocolFeeReceiver;
        for(uint8 i = 0; i < tokensA.length; i ++) {
            initialTokenPairs.push(
                TokenPair(
                    tokensA[i],
                    tokensB[i],
                    swapFees[i]
                )
            );
        }
    }
    // Step 2: Transfer ETH 
    function() external payable {
        require(state == 0, "DexSwapDeployer: WRONG_DEPLOYER_STATE");
        require(msg.sender == owner, "DexSwapDeployer: CALLER_NOT_FEE_TO_SETTER");
        state = 1;
    }
    // Step 3: Deploy DexSwapFactory and all initial pairs
    
    function deploy() public {
        require(state == 1, 'DexSwapDeployer: WRONG_DEPLOYER_STATE');
        DexSwapFactory dexSwapFactory = new DexSwapFactory(address(this));
        emit PairFactoryDeployed(address(dexSwapFactory));
        for(uint8 i = 0; i < initialTokenPairs.length; i ++) {
            address newPair = dexSwapFactory.createPair(initialTokenPairs[i].tokenA, initialTokenPairs[i].tokenB);
            dexSwapFactory.setSwapFee(newPair, initialTokenPairs[i].swapFee);
            emit PairDeployed(address(newPair));
        }
        DexSwapFeeReceiver dexSwapFeeReceiver = new DexSwapFeeReceiver(
            owner, address(dexSwapFactory), WETH, protocolFeeReceiver, owner
        );
        emit FeeReceiverDeployed(address(dexSwapFeeReceiver));
        dexSwapFactory.setFeeTo(address(dexSwapFeeReceiver)); // set Fee To with DexSwapFactory
        
        DexSwapFeeSetter dexSwapFeeSetter = new DexSwapFeeSetter(owner, address(dexSwapFactory));
        emit FeeSetterDeployed(address(dexSwapFeeSetter));
        dexSwapFactory.setFeeToSetter(address(dexSwapFeeSetter)); // set FeeTo Setter with DexSwapFactory to DexSwap FeeSetter Address
        state = 2;
        msg.sender.transfer(address(this).balance);
    }
}

```
#### contracts/Migrations.sol
```javascript
// SPDX-License-Identifier: MIT
pragma solidity >=0.4.25 <0.7.0;

contract Migrations {
  address public owner;
  uint public last_completed_migration;

  modifier restricted() {
    if (msg.sender == owner) _;
  }

  constructor() public {
    owner = msg.sender;
  }

  function setCompleted(uint completed) public restricted {
    last_completed_migration = completed;
  }
}
```
#### contracts/WETH.sol
```javascript
pragma solidity =0.5.16;

contract WETH {
    string public name     = "Wrapped Ether";
    string public symbol   = "WETH";
    uint8  public decimals = 18;

    event  Approval(address indexed src, address indexed guy, uint wad);
    event  Transfer(address indexed src, address indexed dst, uint wad);
    event  Deposit(address indexed dst, uint wad);
    event  Withdrawal(address indexed src, uint wad);

    mapping (address => uint)                       public  balanceOf;
    mapping (address => mapping (address => uint))  public  allowance;

    // function() public payable {
    //     deposit();
    // }
    function deposit() public payable {
        balanceOf[msg.sender] += msg.value;
        emit Deposit(msg.sender, msg.value);
    }
    function withdraw(uint wad) public {
        require(balanceOf[msg.sender] >= wad, "");
        balanceOf[msg.sender] -= wad;
        msg.sender.transfer(wad);
        emit Withdrawal(msg.sender, wad);
    }

    function totalSupply() public view returns (uint) {
        return address(this).balance;
    }

    function approve(address guy, uint wad) public returns (bool) {
        allowance[msg.sender][guy] = wad;
        emit Approval(msg.sender, guy, wad);
        return true;
    }

    function transfer(address dst, uint wad) public returns (bool) {
        return transferFrom(msg.sender, dst, wad);
    }

    function transferFrom(address src, address dst, uint wad)
        public
        returns (bool)
    {
        require(balanceOf[src] >= wad, "");

        if (src != msg.sender && allowance[src][msg.sender] != uint(-1)) {
            require(allowance[src][msg.sender] >= wad, "");
            allowance[src][msg.sender] -= wad;
        }

        balanceOf[src] -= wad;
        balanceOf[dst] += wad;

        emit Transfer(src, dst, wad);

        return true;
    }
}
```
#### Truffle Config
```javascript
const HDWalletProvider = require('@truffle/hdwallet-provider');
const dotenv = require('dotenv');
dotenv.config();

const mnemonic = process.env.MNEMONIC;
const privateKey1 = process.env.PRIVATE_KEY_1;
const privateKey2 = process.env.PRIVATE_KEY_2;
const privateKey3 = process.env.PRIVATE_KEY_3;
console.log(`MNEMONIC: ${process.env.MNEMONIC}`)
console.log(`INFURA_API_KEY: ${process.env.INFURA_API_KEY}`)


module.exports = {
  // Uncommenting the defaults below
  // provides for an easier quick-start with Ganache.
  // You can also follow this format for other networks;
  // see <http://truffleframework.com/docs/advanced/configuration>
  // for more details on how to specify configuration options!
  //
  networks: {
    development: {
      host: "127.0.0.1",
      port: 7545,
      network_id: "*"
    },
    ganache: {
      host: "127.0.0.1",
      port: 8545,
      network_id: "*"
    },
    rinkeby: {
      provider: () => new HDWalletProvider(mnemonic, `https://rinkeby.infura.io/v3/${process.env.INFURA_API_KEY}`),
      network_id: 4,
      gas: 0,
      gasPrice: 2100000001, //2 Gwei,
      skipDryRun: true
    },
    goerli: {
      provider: () => new HDWalletProvider([privateKey1, privateKey2, privateKey3], `https://goerli.infura.io/v3/${process.env.INFURA_API_KEY}`),
      network_id: 5,
      gas: 0,
      gasPrice: 2100000001, //2 Gwei,
      skipDryRun: true
    },
    ropsten: {
      provider: () => new HDWalletProvider([privateKey1, privateKey2, privateKey3], `https://ropsten.infura.io/v3/${process.env.INFURA_API_KEY}`),
      network_id: 3,
      gas: 0,
      gasPrice: 2100000001, //2 Gwei,
      skipDryRun: true
    },
    mumbai: {	
      provider: () => new HDWalletProvider(mnemonic, `https://polygon-mumbai.g.alchemy.com/v2/${process.env.ALCHEMY_API_KEY}`),
      network_id: 80001,
      gasPrice: 2100000001, //2 Gwei,
      skipDryRun: true
    },
    matic: {	
      provider: () => new HDWalletProvider([privateKey1, privateKey2, privateKey3], `https://matic-mainnet.chainstacklabs.com`),
      network_id: 137,
      gas: 0,
      gasPrice: 2100000001, //2 Gwei,
      skipDryRun: true,
      confirmations: 2,
      timeoutBlocks: 200
    }
  },
  plugins: [
    'truffle-plugin-verify'
  ],
  api_keys: {
    etherscan: process.env.ETHERSCAN_API_KEY
  },
  build: {},
  compilers: {
    solc: {
      version: '0.5.16',
      settings: {
        evmVersion: 'istanbul',
      }
    }
  },
  mocha: {
    // timeout: 100000
  },
  solc: {
    optimizer: {
      enabled: true,
      runs: 200
    }
  },
};
```

#### 2_deploy.js
```javascript
const DexSwapFeeReceiver = artifacts.require("DexSwapFeeReceiver");
const DexSwapFeeSetter = artifacts.require("DexSwapFeeSetter");
const DexSwapDeployer = artifacts.require("DexSwapDeployer");
const DexSwapFactory = artifacts.require("DexSwapFactory");
const DexSwapERC20 = artifacts.require("DexSwapERC20");
const WETH = artifacts.require("WETH");

const argValue = (arg, defaultValue) => process.argv.includes(arg) ? process.argv[process.argv.indexOf(arg) + 1] : defaultValue
const network = () => argValue('--network', 'local')

// MATIC MAINNET
const MATIC_WETH = "0x7ceB23fD6bC0adD59E62ac25578270cFf1b9f619";
const xDEXBASE_MUMBAI = ""; //[PoS] <-- see root token tutorial part

// MATIC TESTNET
const MUMBAI_WETH = "0xA6FA4fB5f76172d178d61B04b0ecd319C5d1C0aa";  //[PoS] <-- see root token tutorial part
const xDEXBASE_MUMBAI = "0xBb5e7842a52d54484898B281E0E7F8a73Ee1781c"; //[PoS] <-- see root token tutorial part

module.exports = async (deployer) => {

    const BN = web3.utils.toBN;
    const bnWithDecimals = (number, decimals) => BN(number).mul(BN(10).pow(BN(decimals)));
    const senderAccount = (await web3.eth.getAccounts())[0];

    if (network() === "mumbai") {

        console.log();
        console.log(":: Init DexSwap Deployer");
        await deployer.deploy(DexSwapDeployer, xDEXBASE_MUMBAI, senderAccount, MUMBAI_WETH, [MUMBAI_WETH], [xDEXBASE_MUMBAI], [25]);
        const dexSwapDeployer = await DexSwapDeployer.deployed();


        console.log();
        console.log(":: Start Sending 1 WEI ...");
        await dexSwapDeployer.send(1, {from: senderAccount}); 


        console.log();
        console.log(":: Sent deployment reimbursement");
        await dexSwapDeployer.deploy({from: senderAccount})
        console.log("Deployed dexSwap");


        console.log();
        console.log(":: Deploying Factory");
        await deployer.deploy(DexSwapFactory, senderAccount);
        const DexSwapFactoryInstance = await DexSwapFactory.deployed();
        
        
        console.log();
        console.log(":: Start Deploying DexSwap LP");
        await deployer.deploy(DexSwapERC20);
        const DexSwapLP = await DexSwapERC20.deployed();
        

        console.log(":: Start Deploying FeeReceiver");
        await deployer.deploy(DexSwapFeeReceiver, senderAccount, DexSwapFactoryInstance.address, MUMBAI_WETH, xDEXBASE_MUMBAI, senderAccount);
        const DexSwapFeeReceiverInstance =  await DexSwapFeeReceiver.deployed();
        console.log();


        console.log(":: Start Deploying FeeSetter");
        await deployer.deploy(DexSwapFeeSetter, senderAccount, DexSwapFactoryInstance.address);
        const DexSwapFeeSetterInstance = await DexSwapFeeSetter.deployed();


        console.log();
        console.log(":: Setting Correct FeeSetter in Factory");
        await DexSwapFactoryInstance.setFeeToSetter(DexSwapFeeSetterInstance.address);


        console.log();
        console.log(":: Transfer Ownership FeeReceiver");
        await DexSwapFeeReceiverInstance.transferOwnership(senderAccount);


        console.log();
        console.log(":: Transfer Ownership FeeSetter");
        await DexSwapFeeSetterInstance.transferOwnership(senderAccount);

        
        console.log();
        console.log(":: Updating Protocol FeeReceiver");
        await DexSwapFeeReceiverInstance.changeReceivers(xDEXBASE_MUMBAI, senderAccount, {from: senderAccount});


        console.log();
        console.log("====================================================================");
        console.log(`Deployer Address:`,     dexSwapDeployer.address);
        console.log("====================================================================");

        console.log("====================================================================");
        console.log(`Factory Address:`,      DexSwapFactoryInstance.address);
        console.log("====================================================================");

        console.log("====================================================================");
        console.log(`DexSwap LP Address:`,   DexSwapLP.address);
        console.log("====================================================================");

        console.log("====================================================================");
        console.log(`Fee Setter Address:`,   DexSwapFeeSetterInstance.address);
        console.log("====================================================================");

        console.log("====================================================================");
        console.log(`Fee Receiver Address:`, DexSwapFeeReceiverInstance.address);
        console.log("====================================================================");
        
        console.log("=============================================================================");
        console.log(`Code Hash:`, await DexSwapFactoryInstance.INIT_CODE_PAIR_HASH());
        console.log("=============================================================================");

    } else if (network() === "matic") {


        console.log();
        console.log(":: Init DexSwap Deployer");
        await deployer.deploy(DexSwapDeployer, xDEXSBASE_MATIC, senderAccount, MATIC_WETH , [MATIC_WETH ], [xDEXSBASE_MATIC], [25]);
        const dexSwapDeployer = await DexSwapDeployer.deployed();


        console.log();
        console.log(":: Start Sending 1 WEI ...");
        await dexSwapDeployer.send(1, {from: senderAccount}); 


        console.log();
        console.log(":: Sent deployment reimbursement");
        await dexSwapDeployer.deploy({from: senderAccount})
        console.log("Deployed dexSwap");


        console.log();
        console.log(":: Deploying Factory");
        await deployer.deploy(DexSwapFactory, senderAccount);
        const DexSwapFactoryInstance = await DexSwapFactory.deployed();
        
        
        console.log();
        console.log(":: Start Deploying DexSwap LP");
        await deployer.deploy(DexSwapERC20);
        const DexSwapLP = await DexSwapERC20.deployed();
        

        console.log(":: Start Deploying FeeReceiver");
        await deployer.deploy(DexSwapFeeReceiver, senderAccount, DexSwapFactoryInstance.address, MATIC_WETH, xDEXSBASE_MATIC, senderAccount);
        const DexSwapFeeReceiverInstance =  await DexSwapFeeReceiver.deployed();
        console.log();


        console.log(":: Start Deploying FeeSetter");
        await deployer.deploy(DexSwapFeeSetter, senderAccount, DexSwapFactoryInstance.address);
        const DexSwapFeeSetterInstance = await DexSwapFeeSetter.deployed();


        console.log();
        console.log(":: Setting Correct FeeSetter in Factory");
        await DexSwapFactoryInstance.setFeeToSetter(DexSwapFeeSetterInstance.address);


        console.log();
        console.log(":: Transfer Ownership FeeReceiver");
        await DexSwapFeeReceiverInstance.transferOwnership(senderAccount);


        console.log();
        console.log(":: Transfer Ownership FeeSetter");
        await DexSwapFeeSetterInstance.transferOwnership(senderAccount);

        
        console.log();
        console.log(":: Updating Protocol FeeReceiver");
        await DexSwapFeeReceiverInstance.changeReceivers(xDEXSBASE_MATIC, senderAccount, {from: senderAccount});


        console.log();
        console.log("====================================================================");
        console.log(`Deployer Address:`,     dexSwapDeployer.address);
        console.log("====================================================================");

        console.log("====================================================================");
        console.log(`Factory Address:`,      DexSwapFactoryInstance.address);
        console.log("====================================================================");

        console.log("====================================================================");
        console.log(`DexSwap LP Address:`,   DexSwapLP.address);
        console.log("====================================================================");

        console.log("====================================================================");
        console.log(`Fee Setter Address:`,   DexSwapFeeSetterInstance.address);
        console.log("====================================================================");

        console.log("====================================================================");
        console.log(`Fee Receiver Address:`, DexSwapFeeReceiverInstance.address);
        console.log("====================================================================");
        
        console.log("=============================================================================");
        console.log(`Code Hash:`, await DexSwapFactoryInstance.INIT_CODE_PAIR_HASH());
        console.log("=============================================================================");

    }
};
```    



### Compile and Deploy Smart Contracts
```javascript
truffle compile && truffle migrate --network matic
```
### Result
```javascript
Starting migrations...
======================
> Network name:    'mumbai'
> Network id:      80001
> Block gas limit: 20000000 (0x1312d00)


1_migrations.js
===============

   Deploying 'Migrations'
   ----------------------
   > transaction hash:    0xab78432239430d310867cf96500e2528c9eba839b86583ac2ef05392897f1283
   > Blocks: 3            Seconds: 6
   > contract address:    0xba8074C4D904Fa4B4a79cf4A78d78390A31DD881
   > block number:        16370623
   > block timestamp:     1626338078
   > account:             0xeED588Fc1A009aFf2f4E90b57D4d74B2F56309c5
   > balance:             3.134135931089894289
   > gas used:            130270 (0x1fcde)
   > gas price:           2.1 gwei
   > value sent:          0 ETH
   > total cost:          0.000273567 ETH

   Pausing for 2 confirmations...
   ------------------------------
   > confirmation number: 2 (block: 16370625)

   > Saving migration to chain.
   > Saving artifacts
   -------------------------------------
   > Total cost:         0.000273567 ETH


2_deploy.js
===========

:: Init DexSwap Deployer

   Deploying 'DexSwapDeployer'
   ---------------------------
   > transaction hash:    0x0f6b6d4a8a887b354d001e192502120052211f24c25889c21062fa01cc8c7759
   > Blocks: 5            Seconds: 9
   > contract address:    0xA688D992EC66efa552c9156008EC894824c629DB
   > block number:        16370636
   > block timestamp:     1626338108
   > account:             0xeED588Fc1A009aFf2f4E90b57D4d74B2F56309c5
   > balance:             3.124314464189894289
   > gas used:            4631223 (0x46aab7)
   > gas price:           2.1 gwei
   > value sent:          0 ETH
   > total cost:          0.0097255683 ETH

   Pausing for 2 confirmations...
   ------------------------------
   > confirmation number: 2 (block: 16370641)

:: Start Sending 1 WEI ...

:: Sent deployment reimbursement
Deployed dexSwap

:: Deploying Factory

   Deploying 'DexSwapFactory'
   --------------------------
   > transaction hash:    0xa2e81cc74d250a64160812aa35e037a21c6e3cccca9c9415f7969fb5a8815288
   > Blocks: 4            Seconds: 9
   > contract address:    0x5375d18B43278480D5B79b35580701619FAa5f5d
   > block number:        16370654
   > block timestamp:     1626338144
   > account:             0xeED588Fc1A009aFf2f4E90b57D4d74B2F56309c5
   > balance:             3.105832929089894289
   > gas used:            2690626 (0x290e42)
   > gas price:           2.1 gwei
   > value sent:          0 ETH
   > total cost:          0.0056503146 ETH

   Pausing for 2 confirmations...
   ------------------------------
   > confirmation number: 2 (block: 16370658)

:: Start Deploying DexSwap LP

   Deploying 'DexSwapERC20'
   ------------------------
   > transaction hash:    0xe6a422c8ed299836edf88f36eadc96a940bd2abb87e80690b63488438a400baa
   > Blocks: 3            Seconds: 5
   > contract address:    0xDB3039B37FC1F484355dd4e486D9a8003DD6a5a2
   > block number:        16370664
   > block timestamp:     1626338164
   > account:             0xeED588Fc1A009aFf2f4E90b57D4d74B2F56309c5
   > balance:             3.104678680889894289
   > gas used:            549642 (0x8630a)
   > gas price:           2.1 gwei
   > value sent:          0 ETH
   > total cost:          0.0011542482 ETH

   Pausing for 2 confirmations...
   ------------------------------
   > confirmation number: 2 (block: 16370668)
:: Start Deploying FeeReceiver

   Deploying 'DexSwapFeeReceiver'
   ------------------------------
   > transaction hash:    0xddfb0b0b009ecb4cf28bce301e56f7b5e5ed55416bcd4450b7bd1347ee35ec62
   > Blocks: 2            Seconds: 5
   > contract address:    0xB73F49e9c6666A8Cc05971A950ab63B781c50CC7
   > block number:        16370674
   > block timestamp:     1626338184
   > account:             0xeED588Fc1A009aFf2f4E90b57D4d74B2F56309c5
   > balance:             3.102380140589894289
   > gas used:            1094543 (0x10b38f)
   > gas price:           2.1 gwei
   > value sent:          0 ETH
   > total cost:          0.0022985403 ETH

   Pausing for 2 confirmations...
   ------------------------------
   > confirmation number: 3 (block: 16370679)

:: Start Deploying FeeSetter

   Deploying 'DexSwapFeeSetter'
   ----------------------------
   > transaction hash:    0xf639b7543926a84fea33a657ac977bf79a4901b610071333d3088c74fc26e1a9
   > Blocks: 2            Seconds: 5
   > contract address:    0x6E8355324C012e430072ABCEc597538AF5C58f26
   > block number:        16370684
   > block timestamp:     1626338204
   > account:             0xeED588Fc1A009aFf2f4E90b57D4d74B2F56309c5
   > balance:             3.101455292189894289
   > gas used:            440404 (0x6b854)
   > gas price:           2.1 gwei
   > value sent:          0 ETH
   > total cost:          0.0009248484 ETH

   Pausing for 2 confirmations...
   ------------------------------
   > confirmation number: 1 (block: 16370687)
   > confirmation number: 2 (block: 16370688)

:: Setting Correct FeeSetter in Factory

:: Transfer Ownership FeeReceiver

:: Transfer Ownership FeeSetter

:: Updating Protocol FeeReceiver

====================================================================
Deployer Address: 0xA688D992EC66efa552c9156008EC894824c629DB
====================================================================
====================================================================
Factory Address: 0x5375d18B43278480D5B79b35580701619FAa5f5d
====================================================================
====================================================================
DexSwap LP Address: 0xDB3039B37FC1F484355dd4e486D9a8003DD6a5a2
====================================================================
====================================================================
Fee Setter Address: 0x6E8355324C012e430072ABCEc597538AF5C58f26
====================================================================
====================================================================
Fee Receiver Address: 0xB73F49e9c6666A8Cc05971A950ab63B781c50CC7
====================================================================
=============================================================================
Code Hash: 0x8d5cb477d33ed6bd41c4f92a58f79b1e620735c5408981f4f6aeb73fa189b571
=============================================================================
```


### Verifying Smart Contracts
*  First we need to merging all of our smart contracts dependencies, this can simply done using [Truffle Flattener](https://www.npmjs.com/package/truffle-flattener)

* Copy & Paste the code to flattener .sh
```javascript
npx truffle-flattener contracts/DexSwapFactory.sol > contracts/.flattened/DexSwapFactory.sol
npx truffle-flattener contracts/DexSwapPair.sol > contracts/.flattened/DexSwapPair.sol
npx truffle-flattener contracts/DexSwapERC20.sol > contracts/.flattened/DexSwapERC20.sol
npx truffle-flattener contracts/DexSwapDeployer.sol > contracts/.flattened/DexSwapDeployer.sol
npx truffle-flattener contracts/DexSwapFeeSetter.sol > contracts/.flattened/DexSwapFeeSetter.sol
npx truffle-flattener contracts/DexSwapFeeReceiver.sol > contracts/.flattened/DexSwapFeeReceiver.sol
```

Truffle Flattener concats solidity files from Truffle and Buidler projects with all of their dependencies.

This tool helps you to verify contracts developed with Truffle and Buidler 
on [Etherscan](https://etherscan.io), [Polygonscan](https://polygonscan.com) or debugging them on
[Remix](https://remix.ethereum.org), by merging your files and their
dependencies in the right order.

* Run this in your terminal
```bash
cd contracts
mkdir .flattened
cd ..
./scripts/flattener.sh
```

* Go to [PolygonScan](https://mumbai.polygonscan.com/) <-- we will use mumbai testnet
* Under the contract address, next to the “Contract” tab,
<p align="center">
<img src="https://gateway.pinata.cloud/ipfs/QmWgHDHKzP4LcyutqZBipnMrwYuokP1FZVhjtQv1oZnUv8">
</p>

* Then Select compiler type, compiler version & License
<p align="center">
<img src="https://gateway.pinata.cloud/ipfs/QmZW8KU8fywXPgzcssjHnNb5FKREoXi8pEQH2E5PCuuPSR">
</p>

* Copy & Paste the Solidity contracts from flattener folder
<p align="center">
<img src="https://gateway.pinata.cloud/ipfs/QmcnLHchTXU9yUBfySTh8BrAwBXNxuZENn9NFvmQ5ggVit">
</p>

* If [Constructor Argument](https://docs.soliditylang.org/en/develop/abi-spec.html) didn't fetch automatically we can use [Abi-Hashex](https://abi.hashex.org/#), **todo:** copy the abi, enter the address then copy the abi output from abi-hashex
<p align="center">
<img src="https://gateway.pinata.cloud/ipfs/QmYb3B64Z1hAMmb6wnrm2nhn7vDq7hDG9NQnKUVuhvk9cN">
</p>

* Result
<p align="center">
<img src="https://gateway.pinata.cloud/ipfs/Qme5QUUHYaMxct6BWrC7xWLRLiLieZHxx3NwHVE9KPi9h3">
</p>

<p align="center">
<img src="https://gateway.pinata.cloud/ipfs/QmRF8MYRz5b97HgvWU2BgHcAUYBzGeFQhL8jZqmunQs3Cc">
</p>
