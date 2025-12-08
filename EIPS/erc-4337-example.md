# ERC-4337 账户抽象示例

## 概述

ERC-4337 的主要目标是**账户抽象（Account Abstraction）**，允许用户使用智能合约钱包，而无需修改以太坊协议。

### 核心功能

1. **智能合约钱包**：用户可以使用可编程的智能合约钱包，而不是传统的 EOA 账户
2. **自定义验证逻辑**：支持多签、社交恢复、时间锁等高级功能
3. **批量交易**：在一个 UserOperation 中执行多个操作
4. **Gas 代付（Paymaster）**：允许第三方为用户支付 Gas 费用（这是重要功能之一，但不是唯一目标）

### 核心组件

- **UserOperation**: 用户操作的封装结构
- **Smart Contract Wallet**: 智能合约钱包
- **Bundler**: 打包并提交交易的服务（EOA 账户）
- **Paymaster**: 代付 Gas 费用的合约（可选）
- **EntryPoint**: 处理 UserOperation 的智能合约

## 示例 1: UserOperation 结构

```solidity
struct UserOperation {
    address sender;              // 智能合约钱包地址
    uint256 nonce;               // 防止重放攻击
    bytes initCode;              // 如果钱包未部署，包含部署代码
    bytes callData;              // 要执行的操作数据
    uint256 callGasLimit;        // 调用 Gas 限制
    uint256 verificationGasLimit;// 验证 Gas 限制
    uint256 preVerificationGas; // 预验证 Gas
    uint256 maxFeePerGas;        // 最大 Gas 价格
    uint256 maxPriorityFeePerGas;// 最大优先费用
    bytes paymasterAndData;      // Paymaster 相关数据
    bytes signature;             // 签名数据
}
```

## 示例 2: 简单的智能合约钱包

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "@openzeppelin/contracts/utils/cryptography/ECDSA.sol";

contract SimpleWallet {
    using ECDSA for bytes32;
    
    address public owner;
    uint256 public nonce;
    address public immutable entryPoint;
    
    constructor(address _entryPoint) {
        entryPoint = _entryPoint;
        owner = msg.sender;
    }
    
    function validateUserOp(
        UserOperation calldata userOp,
        bytes32 userOpHash,
        uint256 missingAccountFunds
    ) external returns (uint256 validationData) {
        require(msg.sender == entryPoint, "Only entryPoint");
        require(userOp.nonce == nonce++, "Invalid nonce");
        
        bytes32 hash = keccak256(abi.encodePacked(userOpHash));
        address recovered = hash.recover(userOp.signature);
        require(recovered == owner, "Invalid signature");
        
        if (missingAccountFunds > 0) {
            (bool success, ) = payable(entryPoint).call{value: missingAccountFunds}("");
            require(success, "Failed to pay");
        }
        
        return 0;
    }
    
    function execute(address target, uint256 value, bytes calldata data) external {
        require(msg.sender == entryPoint, "Only entryPoint");
        (bool success, ) = target.call{value: value}(data);
        require(success, "Execution failed");
    }
    
    receive() external payable {}
    function getNonce() external view returns (uint256) { return nonce; }
}
```

## 示例 3: Paymaster 实现

### Paymaster 工作原理

Paymaster 允许第三方为用户支付 Gas 费用。工作流程：

```
1. 用户创建 UserOperation（指定 Paymaster）
2. Bundler（EOA）调用 EntryPoint.handleOps()，支付 Gas
3. EntryPoint 执行：
   a) 验证：Paymaster.validatePaymasterUserOp() - 检查是否愿意支付
   b) 执行：用户钱包.execute() - 执行用户操作
   c) 支付：Paymaster.postOp() - Paymaster 转账给 EntryPoint
4. EntryPoint 补偿 Bundler
```

**资金流向**：
```
第三方账户 → Paymaster 合约（预先充值）
              ↓
           EntryPoint（Paymaster 转账）
              ↓
           Bundler（获得补偿）
```

### Paymaster 合约实现

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract SimplePaymaster {
    address public immutable entryPoint;
    address public owner;
    mapping(address => bool) public whitelistedUsers;
    
    constructor(address _entryPoint) {
        entryPoint = _entryPoint;
        owner = msg.sender;
    }
    
    function addUser(address user) external {
        require(msg.sender == owner, "Only owner");
        whitelistedUsers[user] = true;
    }
    
    // EntryPoint 调用此函数验证是否愿意支付
    function validatePaymasterUserOp(
        UserOperation calldata userOp,
        bytes32 userOpHash,
        uint256 maxCost
    ) external returns (bytes memory context, uint256 validationData) {
        require(msg.sender == entryPoint, "Only entryPoint");
        require(whitelistedUsers[userOp.sender], "User not whitelisted");
        require(address(this).balance >= maxCost, "Insufficient balance");
        return ("", 0);
    }
    
    // EntryPoint 调用此函数，Paymaster 主动支付
    function postOp(
        PostOpMode mode,
        bytes calldata context,
        uint256 actualGasCost
    ) external {
        require(msg.sender == entryPoint, "Only entryPoint");
        payable(entryPoint).transfer(actualGasCost);
    }
    
    receive() external payable {}
}

enum PostOpMode {
    opSucceeded,
    opReverted,
    postOpReverted
}
```

### EntryPoint 如何工作

EntryPoint 是智能合约（不是 EOA），它协调整个流程：

```solidity
contract EntryPoint {
    function handleOps(UserOperation[] calldata ops) external {
        address bundler = msg.sender; // Bundler 是 EOA
        
        for (uint i = 0; i < ops.length; i++) {
            UserOperation memory op = ops[i];
            
            // 1. 验证用户操作
            IAccount(op.sender).validateUserOp(...);
            
            // 2. 如果有 Paymaster，验证 Paymaster
            if (op.paymasterAndData.length > 0) {
                address paymaster = address(bytes20(op.paymasterAndData[0:20]));
                IPaymaster(paymaster).validatePaymasterUserOp(op, ...);
            }
            
            // 3. 执行用户操作
            IAccount(op.sender).execute(...);
            
            // 4. 如果有 Paymaster，让 Paymaster 支付
            if (op.paymasterAndData.length > 0) {
                address paymaster = address(bytes20(op.paymasterAndData[0:20]));
                IPaymaster(paymaster).postOp(PostOpMode.opSucceeded, context, actualGasCost);
                // Paymaster 在 postOp 中转账给 EntryPoint
            }
            
            // 5. EntryPoint 补偿 Bundler
            payable(bundler).transfer(actualGasCost);
        }
    }
}
```

## 示例 4: JavaScript/TypeScript 客户端代码

```typescript
import { ethers } from 'ethers';

async function sendUserOperation(
    walletAddress: string,
    target: string,
    data: string,
    signer: ethers.Signer
) {
    // 1. 获取钱包 nonce
    const wallet = new ethers.Contract(walletAddress, [
        'function getNonce() view returns (uint256)'
    ], signer);
    const nonce = await wallet.getNonce();
    
    // 2. 构建 UserOperation
    const userOp = {
        sender: walletAddress,
        nonce: nonce.toString(),
        initCode: '0x',
        callData: encodeExecuteCall(target, '0', data),
        callGasLimit: '100000',
        verificationGasLimit: '100000',
        preVerificationGas: '50000',
        maxFeePerGas: ethers.utils.parseUnits('20', 'gwei').toString(),
        maxPriorityFeePerGas: ethers.utils.parseUnits('2', 'gwei').toString(),
        paymasterAndData: '0x', // 如果使用 Paymaster
        signature: '0x'
    };
    
    // 3. 计算并签名
    const userOpHash = await entryPoint.getUserOpHash(userOp);
    userOp.signature = await signer.signMessage(ethers.utils.arrayify(userOpHash));
    
    // 4. 发送到 Bundler
    const response = await fetch('https://bundler.example.com/rpc', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
            jsonrpc: '2.0',
            id: 1,
            method: 'eth_sendUserOperation',
            params: [userOp, entryPointAddress]
        })
    });
    
    return (await response.json()).result;
}

function encodeExecuteCall(target: string, value: string, data: string): string {
    const iface = new ethers.utils.Interface([
        'function execute(address target, uint256 value, bytes calldata data)'
    ]);
    return iface.encodeFunctionData('execute', [target, value, data]);
}
```

## 关键要点总结

### 谁支付 Gas？

1. **Bundler（EOA）**：实际支付调用 EntryPoint 的 Gas
2. **Paymaster 合约**：转账给 EntryPoint 补偿 Bundler
3. **用户**：不需要支付 Gas ✅

### 资金流向

```
初始：第三方账户 → Paymaster 合约（充值 10 ETH）

执行：
1. Bundler 支付 Gas（0.001 ETH）
2. Paymaster 转账给 EntryPoint（0.001 ETH）
3. EntryPoint 补偿 Bundler（0.001 ETH）

结果：
- Bundler：余额不变（获得补偿）
- Paymaster：余额减少（9.999 ETH）
- 用户：不需要支付
```

### 关键理解

- EntryPoint 是**智能合约**，不能强制扣除 Paymaster 的费用
- Paymaster **主动同意**支付（validatePaymasterUserOp）并**主动支付**（postOp）
- 如果 Paymaster 拒绝支付，整个交易会回滚

## ERC-4337 的主要价值

### 1. 账户抽象（核心目标）

- **智能合约钱包**：可编程的钱包，支持自定义逻辑
- **多签钱包**：需要多个签名才能执行交易
- **社交恢复**：通过可信联系人恢复钱包
- **时间锁**：延迟执行交易
- **权限管理**：细粒度的权限控制

### 2. Gas 代付（Paymaster）

- **应用代付**：游戏应用为用户支付 Gas
- **新用户引导**：DApp 为新用户的前几笔交易支付 Gas
- **订阅模式**：服务提供商支付 Gas
- **代币支付**：用户用 USDC 支付，Paymaster 转换为 ETH

### 3. 批量交易

- 在一个 UserOperation 中执行多个操作
- 减少 Gas 消耗和交易次数

### 4. 更好的用户体验

- 会话密钥：无需每次签名
- Gasless 交易：用户无需持有 ETH
- 跨链统一体验：相同的钱包地址和逻辑

## 应用场景

- **去中心化社交网络**：CyberConnect 等，用户无需持有 ETH
- **游戏应用**：玩家使用游戏内代币支付 Gas
- **DeFi 应用**：复杂的授权和批量操作
- **企业应用**：多签和权限管理
