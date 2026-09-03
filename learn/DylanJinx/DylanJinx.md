# 学员档案

## 基本信息

| 项目 | 内容 |
| --- | --- |
| GitHub 用户名 | DylanJinx |
| 钱包地址（ETH钱包，用于发奖励） | 0x3A8492819b0C9AB5695D447cbA2532b879d25900 |
| 联系方式（微信 / Telegram / Discord） | Telegram：ddddddddylan / 微信：dylanbuff |
| 是否加入学习交流群 | 是 |

> ⚠️ **钱包地址用于发放奖励，请务必认真填写并反复核对。** 因地址错误导致的奖励无法到账，主办方不承担责任。

## 学习笔记 / 心得

### 第一章：Avalanche 从零入门

Avalanche 是一条已经稳定运行 5 年的 EVM 兼容公链，和以太坊最大的区别在于它的多链架构：

- **Primary Network** 由三条链组成 —— P-Chain（管验证者和质押）、X-Chain（管资产转移）、C-Chain（EVM 合约链，我们写 Solidity 部署的就是这条）。
- **自定义 L1**（原 Subnet）是 Avalanche 真正差异化的地方：企业可以在链的层面自定义验证者集合、Gas 机制、合约部署权限，甚至限定参与者范围。这也是 RWA、支付、金融机构类场景选它的原因。
- 共识用的是 Snowman/Avalanche 系列共识，靠重复随机采样达成概率性终局，出块快、终局时间在秒级。

开发上因为 C-Chain 完全兼容 EVM，Foundry / Hardhat / MetaMask 这套工具链可以直接用，只需要换 RPC 和 chainId：

| | |
| --- | --- |
| 测试网 | Fuji，chainId `43113` |
| RPC | `https://api.avax-test.network/ext/bc/C/rpc` |
| 浏览器 | https://testnet.snowtrace.io |

### 第二章：Vibe Coding 开发第一个 DApp

用 Scaffold-ETH 2 的 Foundry 版本搭了一个 ERC20 应用 `AvaxStudyToken (AVST)`，几点记录：

1. **Scaffold-ETH 的价值在于「合约改完前端自动跟上」**。`yarn deploy` 会跑 `generateTsAbis.js`，把部署地址和 ABI 写进 `packages/nextjs/contracts/deployedContracts.ts`，前端用 `useScaffoldReadContract` / `useScaffoldWriteContract` 按合约名取用，全程有 TypeScript 类型推导，不需要手动拷 ABI。

2. **加 Avalanche 网络只需要改两个文件**：`packages/foundry/foundry.toml` 的 `[rpc_endpoints]` 加 `fuji`，`packages/nextjs/scaffold.config.ts` 的 `targetNetworks` 换成 `chains.avalancheFuji`。

3. **踩的坑**：
   - `create-eth` 要求 Foundry ≥ 1.4.0，本机是 1.2.3，先跑了 `foundryup` 才能创建项目。
   - 前端 lint 报 `react-hooks/purity`：在 render 阶段直接调 `Date.now()` 算 claim 冷却是不允许的（也会造成 SSR 水合不一致），要挪到 `useEffect` 里用 state 存。
   - OpenZeppelin 5.x 的 `Ownable` 构造函数必须显式传 `initialOwner`，和 4.x 默认取 `msg.sender` 不一样。

4. **Vibe Coding 的体感**：让 AI 生成脚手架和样板代码效率很高，但网络配置、版本兼容、lint 规则这类「环境细节」还是要自己盯着，AI 给的代码不一定匹配当前依赖版本。测试是验收 AI 产出的关键手段 —— 这次写了 18 个 Foundry 测试（含 2 个 fuzz），把上限约束和冷却逻辑都覆盖到了。
