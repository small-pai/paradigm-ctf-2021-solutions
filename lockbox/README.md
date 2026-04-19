# Lockbox

## 题目描述

链式依赖型智能合约关卡，合约结构：1 个入口（Entrypoint）+ 5 个关卡（Stage1～5）  
必须严格按顺序解锁：Stage1 → Stage2 → Stage3 → Stage4 → Stage5  
Stage1～3：简单权限 / 状态锁  
Stage4：纯汇编硬编码校验（内存对齐、固定长度数组）  
Stage5：最终校验  
目标：全部通关 → 触发 Entrypoint 返回 Flag  

## 漏洞类型

链式依赖权限绕过（按顺序解锁即可）  
EVM 内存布局 / 数据对齐校验绕过  
汇编级硬编码密码校验绕过  
固定长度数组（bytes32 [6]）ABI 构造  
最终状态校验绕过  

## 漏洞原理

### 1. Stage1 原理
合约内有 `bool public unlocked1`，初始 `false`
`solve()` 把 `unlocked1 = true`
无任何校验
### 2. Stage2 原理
依赖：Stage1 必须已解锁
`solve()` 内部：
```solidity
require(Stage1(unlocked1).unlocked1());
unlocked2 = true;
```
通关
### 3. Stage3 原理
依赖：Stage2 必须已解锁
逻辑同 Stage2：
```solidity
require(Stage2(unlocked2).unlocked2());
unlocked3 = true;
```
### 4. Stage4 原理
函数：`solve(bytes32[6] calldata choices, uint256 pick)`
汇编逐字节校验：
`choices[pick]` 必须是 `"choose"`（右对齐）
数组长度必须严格为 `bytes32[6]`
正确下标：4
错误：`wrong choice!`
### 5. Stage5 原理
依赖：Stage1～4 全部解锁
逻辑：
```solidity
require(Stage1.unlocked1() && Stage2.unlocked2() && Stage3.unlocked3() && Stage4.unlocked4());
unlocked5 = true;
```

## 攻击原理
Stage1：直接调用 solve()  
Stage2：调用 `solve()`（Stage1 已解锁）  
Stage3：调用 `solve()`（Stage2 已解锁）  
Stage4：构造 正确内存布局的 bytes32 [6] 数组（choose 右对齐，下标 4）  
Stage5：调用 `solve()`（Stage1～4 全解锁）  
全部通过 → 从 Entrypoint 获取 Flag  

