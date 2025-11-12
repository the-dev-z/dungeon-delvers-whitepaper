# 🎮 DungeonDelvers 完整技術棧文檔

> **版本**: v2.0.0.0 | **最後更新**: 2025-01-12
> **生態系統**: Base Mainnet GameFi 全棧架構

---

## 📋 目錄

1. [架構概覽](#-架構概覽)
2. [智能合約層](#-1-智能合約層-dungeondelverscontracts)
3. [前端層](#-2-前端層-soulboundsaga)
4. [數據索引層](#-3-數據索引層-dungeon-delvers-subgraph)
5. [後端服務層](#-4-後端服務層-metadata-server)
6. [基礎設施與服務](#-5-基礎設施與服務)
7. [開發工具鏈](#-6-開發工具鏈)
8. [安全與最佳實踐](#-7-安全與最佳實踐)
9. [架構設計模式](#-8-架構設計模式)
10. [性能優化策略](#-9-性能優化策略)
11. [監控指標](#-10-監控指標)

---

## 🏗️ 架構概覽

```
┌─────────────────────────────────────────────────────────────┐
│                     使用者介面層                              │
│   ┌─────────────────────────────────────────────────────┐   │
│   │  前端 (Vite + React + TypeScript)                    │   │
│   │  - Wagmi/Viem: 區塊鏈互動                           │   │
│   │  - TanStack Query: 資料狀態管理                     │   │
│   │  - Zustand: 全局狀態                                │   │
│   └───────────────┬─────────────────┬───────────────────┘   │
│                   │                 │                         │
└───────────────────┼─────────────────┼─────────────────────────┘
                    │                 │
                    ↓                 ↓
        ┌───────────────────┐  ┌──────────────────┐
        │   後端 API        │  │   子圖索引        │
        │   Express + TS    │  │   The Graph      │
        │   Redis Cache     │  │   Goldsky        │
        └────────┬──────────┘  └────────┬─────────┘
                 │                      │
                 ↓                      ↓
        ┌────────────────────────────────────────┐
        │          區塊鏈層 (Base Mainnet)        │
        │  ┌────────────────────────────────┐   │
        │  │  智能合約 (Solidity 0.8.28)    │   │
        │  │  - Hero/Relic NFT System       │   │
        │  │  - DungeonMaster Game Logic    │   │
        │  │  - PlayerVault Economy         │   │
        │  │  - VIPStaking Rewards          │   │
        │  └────────────────────────────────┘   │
        │                                        │
        │  ┌────────────────────────────────┐   │
        │  │  Chainlink VRF V2+             │   │
        │  │  - 隨機數生成                   │   │
        │  │  - 遊戲結果證明                 │   │
        │  └────────────────────────────────┘   │
        │                                        │
        │  ┌────────────────────────────────┐   │
        │  │  Uniswap V3                    │   │
        │  │  - TSOUL/USD Pair              │   │
        │  │  - 流動性管理                   │   │
        │  └────────────────────────────────┘   │
        └────────────────────────────────────────┘
```

### 專案路徑結構

```bash
~/Documents/
├── DungeonDelversContracts/              # 智能合約
├── GitHub/
│   ├── SoulboundSaga/                    # 前端應用
│   ├── dungeon-delvers-subgraph/         # The Graph 子圖
│   └── dungeon-delvers-whitepaper/       # 白皮書文檔
└── dungeon-delvers-metadata-server/      # 後端 API
```

---

## 📦 1. 智能合約層 (DungeonDelversContracts)

### 核心技術框架

| 工具 | 版本 | 用途 |
|------|------|------|
| **Hardhat** | 2.25.0 | 主要開發框架、部署腳本 |
| **Foundry** | Latest | 測試、Gas 優化、Fuzzing |
| **Solidity** | 0.8.28 | 合約開發語言 |
| **OpenZeppelin** | 5.3.0 | 標準合約庫（ERC721、Access Control） |

### 編譯配置

```toml
[profile.default]
solc = "0.8.28"
optimizer = true
optimizer_runs = 200
evm_version = "paris"
via_ir = true                    # 啟用 IR 優化器
```

### 核心依賴包

```json
{
  "dependencies": {
    "@openzeppelin/contracts": "^5.3.0",      // 安全標準庫
    "@chainlink/contracts": "^1.4.0",         // VRF 隨機數
    "@uniswap/v3-core": "^1.0.1"             // DeFi 整合
  },
  "devDependencies": {
    "@nomicfoundation/hardhat-ethers": "^3.0.9",
    "@nomicfoundation/hardhat-toolbox": "^6.0.0",
    "ethers": "^6.14.4",
    "hardhat": "^2.25.0"
  }
}
```

### 部署策略

#### 多階段部署流程

```bash
# Phase 1: 代幣系統
npm run deploy:phase1         # TSOUL, TestUSD1

# Phase 2: 預言機系統
npm run deploy:phase2         # Oracle, VRFConsumer

# Phase 3: 核心系統
npm run deploy:phase3         # DungeonCore, DungeonStorage

# Phase 4: 遊戲模組
npm run deploy:phase4         # Hero, Relic, DungeonMaster, etc.

# 驗證與配置
npm run verify                # Basescan 驗證
npm run setup                 # 合約互連設定
npm run extract-abi           # 提取 ABI
```

#### Gas 價格策略

```javascript
// hardhat.config.js
networks: {
  base: {
    gasPrice: 110000000,      // 固定 0.11 gwei
    accounts: [process.env.PRIVATE_KEY]
  }
}
```

### 合約架構

#### 核心合約 (v2.0.0.0 Base Mainnet)

| 合約名稱 | 地址 | 功能 |
|---------|------|------|
| **Hero** | `0x0949742bffade03016e6e0b6f15de138fe6c5e58` | Hero NFT 鑄造與管理 |
| **Relic** | `0x4ff233dfcb04f27532e0070606310d3dc739c83b` | Relic NFT 鑄造與管理 |
| **Party** | `0xa096b3f5de84bd51f01acd7ce166d4a02c948406` | 隊伍組建系統 |
| **DungeonMaster** | `0xabe0eea017e2689bf85beab6330842010327dbb0` | 探險戰鬥邏輯 |
| **AltarOfAscension** | `0xf686f52e5c6121a1ee85e63d38a2b3a5af33b85a` | Hero 升階系統 |
| **PlayerProfile** | `0x855bb28f51c5d5088c7a1826d87439011d23dcca` | 玩家資料管理 |
| **VIPStaking** | `0xbcbf42f9fbe8efeca66e69d8ce810e12e4221c44` | VIP 質押獎勵 |
| **PlayerVault** | `0x14c062ddb709f7973ef76e037ae109c85a0c4cb0` | 經濟系統與提現 |
| **DungeonCore** | `0xc34fcb2f8d79ec20ebbf6de507ecc8066a8a5ade` | 核心協調器 |
| **SoulShard (TSOUL)** | `0xa5204244859a4f863ece9decb974fa3baba447f7c641ac` | 遊戲代幣 |

### 安全機制

```solidity
// 多層安全防護
contract DungeonCore is Ownable, ReentrancyGuard, Pausable {
    // 1. 權限控制
    modifier onlyAuthorized() {
        require(authorizedContracts[msg.sender], "Unauthorized");
        _;
    }

    // 2. 重入保護
    function criticalOperation() external nonReentrant { ... }

    // 3. 緊急暫停
    function emergencyPause() external onlyOwner {
        _pause();
    }
}
```

---

## 🎨 2. 前端層 (SoulboundSaga)

### 核心技術棧

#### UI 框架

| 技術 | 版本 | 用途 |
|------|------|------|
| **React** | 18.2.0 | UI 組件框架 |
| **TypeScript** | 5.8.3 | 類型安全開發 |
| **Vite** | 6.3.5 | 次世代構建工具 |
| **Tailwind CSS** | 3.4.17 | Utility-first 樣式 |

#### Web3 整合

```json
{
  "viem": "^2.7.9",              // 輕量級 Ethereum 客戶端
  "wagmi": "^2.5.7",             // React Hooks for Ethereum
  "@tanstack/react-query": "^5.8.4",  // 服務端狀態管理
  "zustand": "^4.4.7"            // 客戶端狀態管理
}
```

##### 核心 Hooks 設計

```typescript
// 1. 合約互動 Hook
const { executeTransaction } = useContractTransaction();
await executeTransaction({
  contractCall: {
    address: HERO_ADDRESS,
    abi: HeroABI,
    functionName: 'mintHero',
    args: [rarity]
  },
  description: '鑄造 Hero NFT',
  successMessage: '鑄造成功！',
  errorMessage: '鑄造失敗'
});

// 2. 子圖查詢 Hook (5x 性能提升)
const { data: nfts } = useNftBatchOptimizedV2({
  address: playerAddress,
  enabled: !!playerAddress
});

// 3. 玩家分析 Hook (3x 性能提升)
const { analytics } = usePlayerAnalyticsV2(playerId);
```

#### GraphQL 數據層

```json
{
  "graphql": "^16.11.0",
  "@graphql-codegen/cli": "^5.0.7",
  "@graphql-codegen/typescript": "^4.1.6",
  "@graphql-codegen/typescript-operations": "^4.6.1",
  "@graphql-codegen/typescript-react-query": "^6.1.1"
}
```

##### 自動化代碼生成

```yaml
# codegen.yml
schema: "https://api.goldsky.com/.../v2.0.0.0/gn"
documents: "src/**/*.graphql"
generates:
  src/generated/graphql.ts:
    plugins:
      - typescript
      - typescript-operations
      - typescript-react-query
```

#### UI 組件庫

```json
{
  "@heroicons/react": "^2.2.0",      // Heroicons 圖標
  "lucide-react": "^0.525.0",        // Lucide 圖標
  "chart.js": "^4.5.0",              // 圖表庫
  "react-chartjs-2": "^5.3.0",       // React 包裝器
  "react-hot-toast": "^2.6.0",       // 通知系統
  "clsx": "^2.1.1",                  // 類名工具
  "tailwind-merge": "^3.3.1"         // Tailwind 合併
}
```

### 專案結構

```
src/
├── components/
│   ├── mobile/              # 手機優化組件
│   │   ├── MobileAddress.tsx
│   │   ├── MobileDataCard.tsx
│   │   └── MobileActionMenu.tsx
│   ├── admin/               # 管理面板
│   └── common/              # 通用組件
├── hooks/
│   ├── useContractTransaction.ts     # 合約交互
│   ├── useNftBatchOptimizedV2.ts    # NFT 批次查詢
│   ├── usePlayerAnalyticsV2.ts      # 玩家分析
│   └── useSmartDataLayer.ts         # 智能數據層
├── config/
│   ├── env-contracts.ts     # 環境變數配置
│   └── wagmi.ts             # Wagmi 配置
├── contracts/abi/           # 合約 ABI
├── stores/                  # Zustand 狀態
├── utils/                   # 工具函數
├── types/                   # TypeScript 類型
└── generated/               # GraphQL 自動生成
```

### 配置管理

#### 環境變數系統

```typescript
// src/config/env-contracts.ts
export const contracts = {
  hero: {
    address: import.meta.env.VITE_HERO_ADDRESS as `0x${string}`,
    abi: HeroABI
  },
  relic: {
    address: import.meta.env.VITE_RELIC_ADDRESS as `0x${string}`,
    abi: RelicABI
  }
  // ... 其他合約
} as const;
```

#### Wagmi 配置

```typescript
// src/config/wagmi.ts
import { createConfig, http } from 'wagmi';
import { base } from 'wagmi/chains';

export const config = createConfig({
  chains: [base],
  transports: {
    [base.id]: http(import.meta.env.VITE_BASE_RPC_URL)
  }
});
```

### 開發工具

#### 測試框架

```json
{
  "vitest": "^3.2.4",                       // Vite 原生測試
  "@testing-library/react": "^16.3.0",      // React 組件測試
  "@testing-library/jest-dom": "^6.8.0",    // DOM 斷言
  "@vitest/coverage-v8": "^3.2.4",          // 覆蓋率報告
  "@vitest/ui": "^3.2.4",                   // 測試 UI
  "jsdom": "^22.1.0",                       // DOM 環境
  "fake-indexeddb": "^6.2.2"                // IndexedDB Mock
}
```

#### 代碼品質

```json
{
  "eslint": "^8.55.0",
  "@typescript-eslint/eslint-plugin": "^6.14.0",
  "@typescript-eslint/parser": "^6.14.0",
  "typescript-eslint": "^8.36.0"
}
```

#### 性能分析

```json
{
  "rollup-plugin-visualizer": "^6.0.3",     // Bundle 分析
  "madge": "^8.0.0",                        // 依賴關係圖
  "ts-prune": "^0.10.3"                     // 死代碼檢測
}
```

### 部署配置

#### Vercel 配置

```json
{
  "buildCommand": "npm run build:vercel",
  "outputDirectory": "dist",
  "devCommand": "npm run dev",
  "installCommand": "npm install",
  "framework": "vite",
  "env": {
    "VITE_HERO_ADDRESS": "@hero_address",
    "VITE_BASE_RPC_URL": "@base_rpc_url"
  }
}
```

### 性能指標

- **Bundle 大小**: < 500KB (gzipped)
- **首次內容繪製 (FCP)**: < 1.5s
- **可交互時間 (TTI)**: < 3s
- **Lighthouse 分數**: > 90

---

## 📊 3. 數據索引層 (dungeon-delvers-subgraph)

### The Graph 技術棧

| 技術 | 版本 | 用途 |
|------|------|------|
| **Graph CLI** | 0.97.1 | 子圖開發工具 |
| **Graph TypeScript** | 0.38.1 | AssemblyScript SDK |
| **TypeScript** | 5.3.3 | 類型定義 |

### 核心依賴

```json
{
  "@graphprotocol/graph-cli": "^0.97.1",
  "@graphprotocol/graph-ts": "^0.38.1",
  "typescript": "^5.3.3",
  "jest": "^29.7.0",
  "js-yaml": "^4.1.0"
}
```

### 子圖配置

#### subgraph.yaml

```yaml
specVersion: 1.0.0
schema:
  file: ./schema.graphql
dataSources:
  - kind: ethereum/contract
    name: Hero
    network: base
    source:
      address: "0x0949742bffade03016e6e0b6f15de138fe6c5e58"
      abi: Hero
      startBlock: 37392367
    mapping:
      kind: ethereum/events
      apiVersion: 0.0.7
      language: wasm/assemblyscript
      entities:
        - Hero
        - Player
      abis:
        - name: Hero
          file: ./abis/Hero.json
      eventHandlers:
        - event: Transfer(indexed address,indexed address,indexed uint256)
          handler: handleTransfer
        - event: HeroMinted(indexed uint256,indexed address,uint8)
          handler: handleHeroMinted
      file: ./src/mappings/hero.ts
```

### GraphQL Schema 設計

```graphql
# 玩家實體
type Player @entity {
  id: ID!                                    # 玩家地址
  heros: [Hero!]! @derivedFrom(field: "owner")
  relics: [Relic!]! @derivedFrom(field: "owner")
  parties: [Party!]! @derivedFrom(field: "leader")
  expeditions: [Expedition!]! @derivedFrom(field: "player")
  totalHeroes: BigInt!
  totalRelics: BigInt!
  totalExpeditions: BigInt!
  successfulExpeditions: BigInt!
  totalRewardsEarned: BigInt!
  vipLevel: BigInt!
  registeredAt: BigInt!
  lastActiveAt: BigInt!
}

# Hero NFT 實體
type Hero @entity {
  id: ID!                                    # tokenId
  tokenId: BigInt!
  owner: Player!
  rarity: String!                            # Common, Rare, Epic, Legendary
  power: BigInt!
  level: BigInt!
  experience: BigInt!
  isAscended: Boolean!
  mintedAt: BigInt!
  lastUpgradedAt: BigInt
  transferHistory: [Transfer!]! @derivedFrom(field: "hero")
}

# 探險記錄實體
type Expedition @entity {
  id: ID!                                    # txHash-logIndex
  player: Player!
  party: Party!
  dungeonLevel: BigInt!
  success: Boolean!
  reward: BigInt!
  timestamp: BigInt!
  blockNumber: BigInt!
}

# VRF 請求實體
type VRFRequest @entity {
  id: ID!                                    # requestId
  requestType: String!                       # MINT_HERO, ASCEND, EXPEDITION
  requester: Player!
  status: String!                            # PENDING, FULFILLED, FAILED
  randomness: BigInt
  createdAt: BigInt!
  fulfilledAt: BigInt
}
```

### 事件處理器

```typescript
// src/mappings/hero.ts
import { Hero, Player, Transfer } from "../generated/schema";
import { Transfer as TransferEvent, HeroMinted } from "../generated/Hero/Hero";

export function handleHeroMinted(event: HeroMinted): void {
  let hero = new Hero(event.params.tokenId.toString());
  hero.tokenId = event.params.tokenId;
  hero.owner = event.params.owner.toHexString();
  hero.rarity = getRarityString(event.params.rarity);
  hero.power = BigInt.fromI32(0);
  hero.level = BigInt.fromI32(1);
  hero.experience = BigInt.fromI32(0);
  hero.isAscended = false;
  hero.mintedAt = event.block.timestamp;
  hero.save();

  // 更新玩家統計
  let player = Player.load(event.params.owner.toHexString());
  if (player == null) {
    player = new Player(event.params.owner.toHexString());
    player.totalHeroes = BigInt.fromI32(0);
    player.totalRelics = BigInt.fromI32(0);
    player.registeredAt = event.block.timestamp;
  }
  player.totalHeroes = player.totalHeroes.plus(BigInt.fromI32(1));
  player.lastActiveAt = event.block.timestamp;
  player.save();
}
```

### 部署端點

#### Goldsky (主要生產環境)

```bash
# 部署到 Goldsky
goldsky subgraph deploy dungeon-delvers/v2.0.0.0 --path ./build

# 查詢端點
https://api.goldsky.com/api/public/project_cmfgfjap16zr601wn6pny06zt/subgraphs/dungeon-delvers/v2.0.0.0/gn
```

#### The Graph Studio (備用)

```bash
# 部署到 Studio
graph deploy dungeon-delvers---base \
  --access-token $GRAPH_ACCESS_TOKEN \
  --node https://api.studio.thegraph.com/deploy/

# 查詢端點
https://api.studio.thegraph.com/query/115633/dungeon-delvers---base/v2.0.0.0
```

### 常用查詢範例

#### 查詢玩家資產

```graphql
query PlayerAssets($address: String!) {
  player(id: $address) {
    id
    totalHeroes
    totalRelics
    heros(first: 100, orderBy: power, orderDirection: desc) {
      id
      tokenId
      rarity
      power
      level
      isAscended
    }
    relics(first: 100) {
      id
      tokenId
      rarity
      capacity
    }
    parties {
      id
      totalPower
      partyRarity
      members {
        tokenId
        rarity
      }
    }
  }
}
```

#### 查詢遊戲統計

```graphql
query GameStatistics {
  expeditions(
    first: 1000
    orderBy: timestamp
    orderDirection: desc
    where: { success: true }
  ) {
    id
    player { id }
    dungeonLevel
    reward
    timestamp
  }

  heroes(first: 10, orderBy: power, orderDirection: desc) {
    tokenId
    owner { id }
    rarity
    power
    level
  }
}
```

### 性能優化

#### 索引策略

```graphql
type Hero @entity {
  id: ID!
  tokenId: BigInt! @index
  owner: Player! @index
  rarity: String! @index
  power: BigInt! @index
  level: BigInt!
}
```

#### 分頁查詢

```typescript
// 前端實現分頁
const PAGE_SIZE = 50;

const { data } = useQuery({
  queryKey: ['heroes', page],
  queryFn: async () => {
    const skip = page * PAGE_SIZE;
    return fetchHeroes({ first: PAGE_SIZE, skip });
  }
});
```

### 監控指標

- **同步延遲**: < 10 blocks
- **查詢響應時間**: < 500ms (p95)
- **錯誤率**: < 0.1%
- **索引完整性**: 100%

---

## 🖥️ 4. 後端服務層 (metadata-server)

### 核心技術棧

| 技術 | 版本 | 用途 |
|------|------|------|
| **Node.js** | >= 18.0.0 | 執行環境 |
| **Express** | 4.18.2 | Web 框架 |
| **Redis** | (ioredis 5.7.0) | 分布式快取 |
| **Winston** | 3.17.0 | 日誌系統 |

### 核心依賴

```json
{
  "express": "^4.18.2",              // Web 框架
  "dotenv": "^16.3.1",               // 環境變數
  "cors": "^2.8.5",                  // 跨域處理
  "compression": "^1.7.4",           // 響應壓縮
  "helmet": "^7.1.0",                // 安全頭部
  "ioredis": "^5.7.0",               // Redis 客戶端
  "node-cache": "^5.1.2",            // 內存快取
  "rate-limiter-flexible": "^4.0.1", // 速率限制
  "winston": "^3.17.0",              // 結構化日誌
  "morgan": "^1.10.0",               // HTTP 日誌
  "axios": "^1.6.0"                  // HTTP 客戶端
}
```

### 專案結構

```
src/
├── index.js                 # 應用入口
├── routes/
│   ├── metadata.js          # NFT 元數據 API
│   ├── images.js            # 圖片生成 API
│   └── health.js            # 健康檢查
├── services/
│   ├── cacheService.js      # 快取管理
│   ├── imageService.js      # 圖片生成
│   └── contractService.js   # 合約查詢
├── middleware/
│   ├── rateLimiter.js       # 速率限制
│   ├── errorHandler.js      # 錯誤處理
│   └── logger.js            # 日誌中間件
├── utils/
│   ├── logger.js            # Winston 配置
│   └── redis.js             # Redis 連接
└── config/
    └── constants.js         # 常量配置
```

### API 端點設計

#### 1. NFT 元數據 API

```javascript
/**
 * GET /api/metadata/hero/:tokenId
 * 獲取 Hero NFT 元數據
 */
app.get('/api/metadata/hero/:tokenId', async (req, res) => {
  const { tokenId } = req.params;

  // 1. 檢查快取
  const cached = await cacheService.get(`hero:${tokenId}`);
  if (cached) return res.json(cached);

  // 2. 查詢合約
  const metadata = await contractService.getHeroMetadata(tokenId);

  // 3. 設置快取 (1 小時)
  await cacheService.set(`hero:${tokenId}`, metadata, 3600);

  res.json(metadata);
});

/**
 * GET /api/metadata/relic/:tokenId
 * 獲取 Relic NFT 元數據
 */
app.get('/api/metadata/relic/:tokenId', async (req, res) => {
  // 類似實現
});
```

#### 2. 圖片生成 API

```javascript
/**
 * GET /api/images/hero/:tokenId
 * 動態生成 Hero NFT 圖片
 */
app.get('/api/images/hero/:tokenId', async (req, res) => {
  const { tokenId } = req.params;

  // 1. 獲取 Hero 數據
  const hero = await contractService.getHeroData(tokenId);

  // 2. 生成圖片
  const imageBuffer = await imageService.generateHeroImage(hero);

  // 3. 設置快取頭
  res.set({
    'Content-Type': 'image/png',
    'Cache-Control': 'public, max-age=86400'
  });

  res.send(imageBuffer);
});
```

#### 3. 健康檢查 API

```javascript
/**
 * GET /health
 * 服務健康狀態
 */
app.get('/health', async (req, res) => {
  const status = {
    server: 'OK',
    redis: await checkRedisConnection(),
    uptime: process.uptime(),
    memory: process.memoryUsage(),
    timestamp: new Date().toISOString()
  };

  res.json(status);
});
```

### 快取架構

#### 多層快取策略

```javascript
class CacheService {
  constructor() {
    this.memoryCache = new NodeCache({ stdTTL: 300 }); // L1: 5分鐘
    this.redis = new Redis({
      host: process.env.REDIS_HOST,
      port: process.env.REDIS_PORT,
      password: process.env.REDIS_PASSWORD
    });
  }

  async get(key) {
    // 1. 檢查內存快取
    let value = this.memoryCache.get(key);
    if (value) {
      logger.debug('Cache hit (memory)', { key });
      return value;
    }

    // 2. 檢查 Redis 快取
    const redisValue = await this.redis.get(key);
    if (redisValue) {
      value = JSON.parse(redisValue);
      this.memoryCache.set(key, value);
      logger.debug('Cache hit (redis)', { key });
      return value;
    }

    logger.debug('Cache miss', { key });
    return null;
  }

  async set(key, value, ttl = 3600) {
    this.memoryCache.set(key, value, ttl);
    await this.redis.setex(key, ttl, JSON.stringify(value));
  }
}
```

### 安全中間件

#### 速率限制

```javascript
const { RateLimiterRedis } = require('rate-limiter-flexible');

const rateLimiter = new RateLimiterRedis({
  storeClient: redisClient,
  points: 100,          // 100 次請求
  duration: 60,         // 每 60 秒
  blockDuration: 300    // 封鎖 5 分鐘
});

app.use(async (req, res, next) => {
  try {
    await rateLimiter.consume(req.ip);
    next();
  } catch (error) {
    res.status(429).json({ error: 'Too Many Requests' });
  }
});
```

#### 安全頭部

```javascript
const helmet = require('helmet');

app.use(helmet({
  contentSecurityPolicy: {
    directives: {
      defaultSrc: ["'self'"],
      styleSrc: ["'self'", "'unsafe-inline'"],
      scriptSrc: ["'self'"],
      imgSrc: ["'self'", 'data:', 'https:']
    }
  },
  hsts: {
    maxAge: 31536000,
    includeSubDomains: true,
    preload: true
  }
}));
```

### 日誌系統

#### Winston 配置

```javascript
const winston = require('winston');

const logger = winston.createLogger({
  level: process.env.LOG_LEVEL || 'info',
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.errors({ stack: true }),
    winston.format.json()
  ),
  transports: [
    // 1. Console 輸出
    new winston.transports.Console({
      format: winston.format.simple()
    }),
    // 2. 錯誤日誌文件
    new winston.transports.File({
      filename: 'logs/error.log',
      level: 'error',
      maxsize: 5242880,  // 5MB
      maxFiles: 5
    }),
    // 3. 綜合日誌文件
    new winston.transports.File({
      filename: 'logs/combined.log',
      maxsize: 5242880,
      maxFiles: 10
    })
  ]
});

// HTTP 請求日誌
app.use(morgan('combined', {
  stream: {
    write: (message) => logger.info(message.trim())
  }
}));
```

### 部署配置 (Render.com)

#### render.yaml

```yaml
services:
  - type: web
    name: dungeon-delvers-metadata-server
    env: node
    plan: starter
    buildCommand: npm install
    startCommand: npm run start
    envVars:
      - key: NODE_ENV
        value: production
      - key: PORT
        value: 10000
      - key: REDIS_HOST
        sync: false
      - key: REDIS_PASSWORD
        sync: false
      - key: BASE_RPC_URL
        sync: false
    healthCheckPath: /health
    autoDeploy: true
```

#### 記憶體優化

```json
{
  "scripts": {
    "start": "node --max-old-space-size=500 --optimize-for-size --gc-interval=50 src/index.js"
  }
}
```

### 監控指標

- **API 響應時間**: < 200ms (p95)
- **快取命中率**: > 80%
- **錯誤率**: < 0.5%
- **記憶體使用**: < 450MB
- **請求頻率**: ~1000 req/min

---

## 🔗 5. 基礎設施與服務

### 區塊鏈網路

#### Base Mainnet 配置

```javascript
{
  chainId: 8453,
  name: 'Base',
  network: 'base',
  nativeCurrency: {
    name: 'Ether',
    symbol: 'ETH',
    decimals: 18
  },
  rpcUrls: {
    default: {
      http: ['https://mainnet.base.org'],
      webSocket: ['wss://mainnet.base.org']
    },
    public: {
      http: ['https://mainnet.base.org']
    },
    alchemy: {
      http: ['https://base-mainnet.g.alchemy.com/v2/{API_KEY}']
    }
  },
  blockExplorers: {
    default: {
      name: 'Basescan',
      url: 'https://basescan.org'
    }
  },
  contracts: {
    multicall3: {
      address: '0xca11bde05977b3631167028862be2a173976ca11',
      blockCreated: 5022
    }
  }
}
```

#### Base Sepolia (測試網)

```javascript
{
  chainId: 84532,
  name: 'Base Sepolia',
  network: 'base-sepolia',
  testnet: true,
  rpcUrls: {
    default: {
      http: ['https://sepolia.base.org']
    }
  },
  blockExplorers: {
    default: {
      name: 'Basescan',
      url: 'https://sepolia.basescan.org'
    }
  }
}
```

### Chainlink VRF 整合

#### VRF V2+ 配置

```solidity
// contracts/current/oracles/VRFConsumerV2Plus.sol
contract VRFConsumerV2Plus {
    address constant VRF_COORDINATOR = 0xd5d517abe5cf79b7e95ec98db0f0277788aff634;
    bytes32 constant KEY_HASH = 0x00b8...73ccab;  // Base VRF Key Hash
    uint256 constant SUBSCRIPTION_ID = 38276839386691214737119618087264095086513504326345898083808435856639025322726;

    uint32 constant CALLBACK_GAS_LIMIT = 500000;
    uint16 constant REQUEST_CONFIRMATIONS = 3;
    uint32 constant NUM_WORDS = 1;

    function requestRandomWords() external returns (uint256 requestId) {
        requestId = COORDINATOR.requestRandomWords(
            KEY_HASH,
            SUBSCRIPTION_ID,
            REQUEST_CONFIRMATIONS,
            CALLBACK_GAS_LIMIT,
            NUM_WORDS
        );
    }

    function fulfillRandomWords(
        uint256 requestId,
        uint256[] memory randomWords
    ) internal override {
        // 處理隨機數回調
    }
}
```

#### VRF 使用場景

| 合約 | 用途 | Gas Limit |
|------|------|-----------|
| **Hero** | 鑄造時決定稀有度 | 300,000 |
| **Relic** | 生成隨機屬性 | 250,000 |
| **AltarOfAscension** | 升階成功率判定 | 350,000 |
| **DungeonMaster** | 探險戰鬥結果 | 500,000 |

### Uniswap V3 整合

#### 流動性池配置

```javascript
{
  pairAddress: '0xfab0c3ed7dbf35f919b81c415ae2e63192c55e50',
  token0: {
    address: '0x12ea4682604ba45ecbd974fa3baba447f7c641ac',  // TestUSD1
    symbol: 'USD1',
    decimals: 18
  },
  token1: {
    address: '0xa5204244859a4f863ece9decb974fa3baba447f7c641ac',  // TSOUL
    symbol: 'TSOUL',
    decimals: 18
  },
  fee: 3000,  // 0.3%
  tickSpacing: 60
}
```

#### 價格查詢

```typescript
import { useUniswapV3Price } from '@/hooks/useUniswapV3Price';

const { price, liquidity } = useUniswapV3Price({
  pairAddress: UNISWAP_V3_PAIR,
  token0: TSOUL_ADDRESS,
  token1: USD1_ADDRESS
});

console.log(`1 TSOUL = ${price} USD1`);
```

### RPC 提供商

#### 主要 RPC 端點

```bash
# 1. Coinbase Cloud (優先)
https://api.developer.coinbase.com/rpc/v1/base/{PROJECT_ID}

# 2. Alchemy (備用)
https://base-mainnet.g.alchemy.com/v2/{API_KEY}

# 3. Base 公共 RPC (開發)
https://mainnet.base.org
```

#### RPC 負載均衡

```typescript
const rpcProviders = [
  { url: process.env.COINBASE_RPC_URL, priority: 1, weight: 70 },
  { url: process.env.ALCHEMY_RPC_URL, priority: 2, weight: 20 },
  { url: 'https://mainnet.base.org', priority: 3, weight: 10 }
];

function selectRPC() {
  const rand = Math.random() * 100;
  let cumulative = 0;

  for (const provider of rpcProviders) {
    cumulative += provider.weight;
    if (rand < cumulative) return provider.url;
  }
}
```

---

## 🛠️ 6. 開發工具鏈

### 版本控制

#### Git 工作流程

```bash
# 主分支策略
main          # 生產環境 (v2.0.0.0)
develop       # 開發環境
feature/*     # 功能分支
hotfix/*      # 緊急修復

# 標準工作流
git checkout -b feature/new-hero-system develop
# ... 開發功能
git commit -m "feat: add new hero minting system"
git push origin feature/new-hero-system
# ... 創建 Pull Request
```

#### 提交規範 (Conventional Commits)

```bash
feat:     新功能
fix:      Bug 修復
docs:     文檔更新
style:    代碼格式（不影響功能）
refactor: 重構
perf:     性能優化
test:     測試相關
chore:    構建工具、輔助工具

# 範例
git commit -m "feat(hero): add legendary hero minting"
git commit -m "fix(dungeon): resolve expedition reward calculation"
git commit -m "perf(subgraph): optimize player query indexing"
```

### CI/CD 流程

#### 前端 (Vercel)

```yaml
name: Frontend CI/CD

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'

      - name: Install dependencies
        run: npm ci

      - name: Type check
        run: npm run type-check

      - name: Lint
        run: npm run lint

      - name: Test
        run: npm run test:run

      - name: Build
        run: npm run build
        env:
          VITE_HERO_ADDRESS: ${{ secrets.VITE_HERO_ADDRESS }}

      - name: Deploy to Vercel
        if: github.ref == 'refs/heads/main'
        run: vercel --prod
        env:
          VERCEL_TOKEN: ${{ secrets.VERCEL_TOKEN }}
```

#### 後端 (Render.com)

```yaml
name: Backend CI/CD

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'

      - name: Install dependencies
        run: npm ci

      - name: Run tests
        run: npm test

      - name: Deploy to Render
        uses: johnbeynon/render-deploy-action@v0.0.8
        with:
          service-id: ${{ secrets.RENDER_SERVICE_ID }}
          api-key: ${{ secrets.RENDER_API_KEY }}
```

#### 子圖 (Goldsky)

```yaml
name: Subgraph CI/CD

on:
  push:
    branches: [main]
    paths:
      - 'subgraph/**'
      - 'schema.graphql'

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Install Graph CLI
        run: npm install -g @graphprotocol/graph-cli

      - name: Codegen
        run: npm run codegen

      - name: Build
        run: npm run build

      - name: Deploy to Goldsky
        run: |
          goldsky subgraph deploy dungeon-delvers/v2.0.0.0 \
            --path ./build \
            --token ${{ secrets.GOLDSKY_TOKEN }}
```

### 測試策略

#### 前端測試

```typescript
// vitest.config.ts
import { defineConfig } from 'vitest/config';

export default defineConfig({
  test: {
    globals: true,
    environment: 'jsdom',
    setupFiles: ['./src/test/setup.ts'],
    coverage: {
      provider: 'v8',
      reporter: ['text', 'json', 'html'],
      exclude: [
        'node_modules/',
        'src/test/',
        '**/*.d.ts',
        '**/*.config.*',
        '**/generated/**'
      ]
    }
  }
});
```

##### 單元測試範例

```typescript
// src/hooks/useContractTransaction.test.ts
import { renderHook, waitFor } from '@testing-library/react';
import { describe, it, expect, vi } from 'vitest';
import { useContractTransaction } from './useContractTransaction';

describe('useContractTransaction', () => {
  it('should execute transaction successfully', async () => {
    const { result } = renderHook(() => useContractTransaction());

    await act(async () => {
      await result.current.executeTransaction({
        contractCall: {
          address: HERO_ADDRESS,
          abi: HeroABI,
          functionName: 'mintHero',
          args: [1]
        },
        description: 'Mint Hero'
      });
    });

    expect(result.current.isSuccess).toBe(true);
  });
});
```

#### 智能合約測試

```solidity
// test/Hero.t.sol (Foundry)
pragma solidity ^0.8.28;

import "forge-std/Test.sol";
import "../contracts/current/nft/Hero.sol";

contract HeroTest is Test {
    Hero public hero;
    address public player = address(0x1);

    function setUp() public {
        hero = new Hero();
        vm.deal(player, 10 ether);
    }

    function testMintHero() public {
        vm.startPrank(player);

        uint256 tokenId = hero.mintHero{value: 0.1 ether}(1);

        assertEq(hero.ownerOf(tokenId), player);
        assertEq(hero.balanceOf(player), 1);

        vm.stopPrank();
    }

    function testCannotMintWithInsufficientPayment() public {
        vm.startPrank(player);

        vm.expectRevert("Insufficient payment");
        hero.mintHero{value: 0.01 ether}(1);

        vm.stopPrank();
    }
}
```

```bash
# 執行 Foundry 測試
forge test -vv

# Gas 報告
forge test --gas-report

# 覆蓋率報告
forge coverage
```

#### 後端測試

```javascript
// tests/api/metadata.test.js
const request = require('supertest');
const app = require('../../src/index');

describe('Metadata API', () => {
  describe('GET /api/metadata/hero/:tokenId', () => {
    it('should return hero metadata', async () => {
      const response = await request(app)
        .get('/api/metadata/hero/1')
        .expect(200)
        .expect('Content-Type', /json/);

      expect(response.body).toHaveProperty('name');
      expect(response.body).toHaveProperty('image');
      expect(response.body).toHaveProperty('attributes');
    });

    it('should return 404 for non-existent hero', async () => {
      await request(app)
        .get('/api/metadata/hero/999999')
        .expect(404);
    });
  });
});
```

### 代碼品質工具

#### ESLint 配置

```javascript
// .eslintrc.cjs
module.exports = {
  root: true,
  env: { browser: true, es2020: true, node: true },
  extends: [
    'eslint:recommended',
    'plugin:@typescript-eslint/recommended',
    'plugin:react-hooks/recommended',
  ],
  ignorePatterns: ['dist', '.eslintrc.cjs', 'generated'],
  parser: '@typescript-eslint/parser',
  plugins: ['react-refresh'],
  rules: {
    '@typescript-eslint/no-explicit-any': 'warn',
    '@typescript-eslint/no-unused-vars': ['error', {
      argsIgnorePattern: '^_',
      varsIgnorePattern: '^_'
    }],
    'react-hooks/rules-of-hooks': 'error',
    'react-hooks/exhaustive-deps': 'warn'
  }
};
```

#### TypeScript 嚴格模式

```json
// tsconfig.json
{
  "compilerOptions": {
    "target": "ES2020",
    "useDefineForClassFields": true,
    "lib": ["ES2020", "DOM", "DOM.Iterable"],
    "module": "ESNext",
    "skipLibCheck": true,
    "strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noFallthroughCasesInSwitch": true,
    "noUncheckedIndexedAccess": true,
    "moduleResolution": "bundler",
    "allowImportingTsExtensions": true,
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true,
    "jsx": "react-jsx"
  }
}
```

---

## 🛡️ 7. 安全與最佳實踐

### 智能合約安全

#### OpenZeppelin 標準實現

```solidity
import "@openzeppelin/contracts/token/ERC721/ERC721.sol";
import "@openzeppelin/contracts/access/Ownable.sol";
import "@openzeppelin/contracts/security/ReentrancyGuard.sol";
import "@openzeppelin/contracts/security/Pausable.sol";

contract Hero is ERC721, Ownable, ReentrancyGuard, Pausable {
    // 1. 重入保護
    function mintHero(uint8 rarity) external payable nonReentrant whenNotPaused {
        require(msg.value >= mintPrice, "Insufficient payment");
        // 鑄造邏輯
    }

    // 2. 緊急暫停
    function emergencyPause() external onlyOwner {
        _pause();
    }

    // 3. 安全提現
    function withdraw() external onlyOwner {
        uint256 balance = address(this).balance;
        require(balance > 0, "No balance");

        (bool success, ) = owner().call{value: balance}("");
        require(success, "Transfer failed");
    }
}
```

#### Checks-Effects-Interactions 模式

```solidity
function claimRewards() external nonReentrant {
    uint256 amount = pendingRewards[msg.sender];

    // 1. Checks
    require(amount > 0, "No rewards");
    require(address(this).balance >= amount, "Insufficient balance");

    // 2. Effects
    pendingRewards[msg.sender] = 0;
    totalClaimed += amount;

    // 3. Interactions
    (bool success, ) = msg.sender.call{value: amount}("");
    require(success, "Transfer failed");

    emit RewardsClaimed(msg.sender, amount);
}
```

#### 整數溢出保護

```solidity
// Solidity 0.8+ 內建溢出檢查
function addExperience(uint256 heroId, uint256 amount) external {
    Hero storage hero = heroes[heroId];

    // 自動檢查溢出
    hero.experience += amount;

    // 明確檢查上限
    require(hero.experience <= MAX_EXPERIENCE, "Max experience reached");
}
```

### 前端安全

#### 環境變數管理

```bash
# .env.example（公開）
VITE_HERO_ADDRESS=
VITE_RELIC_ADDRESS=
VITE_BASE_RPC_URL=
VITE_SUBGRAPH_URL=

# .env（Git ignored）
VITE_HERO_ADDRESS=0x0949742bffade03016e6e0b6f15de138fe6c5e58
VITE_BASE_RPC_URL=https://api.developer.coinbase.com/rpc/v1/base/xxx
```

```typescript
// 類型安全的環境變數
interface ImportMetaEnv {
  readonly VITE_HERO_ADDRESS: `0x${string}`;
  readonly VITE_RELIC_ADDRESS: `0x${string}`;
  readonly VITE_BASE_RPC_URL: string;
  readonly VITE_SUBGRAPH_URL: string;
}

// 驗證環境變數
function validateEnv(): void {
  const required = [
    'VITE_HERO_ADDRESS',
    'VITE_RELIC_ADDRESS',
    'VITE_BASE_RPC_URL'
  ];

  for (const key of required) {
    if (!import.meta.env[key]) {
      throw new Error(`Missing required environment variable: ${key}`);
    }
  }
}

validateEnv();
```

#### XSS 防護

```typescript
// 使用 DOMPurify 清理用戶輸入
import DOMPurify from 'dompurify';

function DisplayUserName({ name }: { name: string }) {
  const sanitizedName = DOMPurify.sanitize(name);

  return (
    <div dangerouslySetInnerHTML={{ __html: sanitizedName }} />
  );
}

// 或直接使用 React 的自動轉義
function DisplayUserName({ name }: { name: string }) {
  return <div>{name}</div>;  // React 自動轉義
}
```

#### CSRF 防護

```typescript
// 使用 Wagmi 的簽名驗證
async function authenticateUser(address: string) {
  const message = `Sign this message to authenticate: ${Date.now()}`;

  const signature = await signMessage({
    message
  });

  // 發送到後端驗證
  const response = await fetch('/api/auth', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ address, message, signature })
  });

  return response.json();
}
```

### 後端安全

#### 輸入驗證

```javascript
const Joi = require('joi');

// 定義驗證 Schema
const heroMetadataSchema = Joi.object({
  tokenId: Joi.number().integer().min(1).required()
});

// 中間件
function validateRequest(schema) {
  return (req, res, next) => {
    const { error } = schema.validate(req.params);

    if (error) {
      return res.status(400).json({
        error: 'Validation failed',
        details: error.details
      });
    }

    next();
  };
}

// 使用
app.get(
  '/api/metadata/hero/:tokenId',
  validateRequest(heroMetadataSchema),
  async (req, res) => {
    // 處理請求
  }
);
```

#### SQL 注入防護

```javascript
// 使用參數化查詢
async function getPlayerData(address) {
  // ❌ 錯誤：SQL 注入風險
  const query = `SELECT * FROM players WHERE address = '${address}'`;

  // ✅ 正確：參數化查詢
  const query = 'SELECT * FROM players WHERE address = $1';
  const result = await db.query(query, [address]);

  return result.rows[0];
}
```

#### API 密鑰管理

```javascript
// 環境變數分離
const config = {
  development: {
    rpcUrl: process.env.DEV_BASE_RPC_URL,
    apiKey: process.env.DEV_API_KEY
  },
  production: {
    rpcUrl: process.env.PROD_BASE_RPC_URL,
    apiKey: process.env.PROD_API_KEY
  }
};

const env = process.env.NODE_ENV || 'development';
module.exports = config[env];
```

### 私鑰管理

#### 安全檢查清單

```bash
#!/bin/bash
# scripts/security-check.sh

echo "🔍 執行私鑰安全掃描..."

# 1. 檢查硬編碼私鑰
echo "檢查硬編碼私鑰..."
find . -type f \( -name "*.ts" -o -name "*.js" -o -name "*.sol" \) \
  -not -path "*/node_modules/*" \
  -not -path "*/dist/*" \
  | xargs grep -n "0x[0-9a-fA-F]\{64\}" \
  && echo "⚠️  發現疑似私鑰！" || echo "✅ 未發現硬編碼私鑰"

# 2. 檢查 .env 文件是否被追蹤
echo "檢查環境變數文件..."
git ls-files | grep -E "\.env$|\.env\.local$" \
  && echo "❌ .env 文件被 Git 追蹤！" || echo "✅ .env 文件未被追蹤"

# 3. 檢查註解中的敏感資訊
echo "檢查註解中的敏感資訊..."
grep -r -i "private.*key\|secret.*key\|mnemonic" . \
  --exclude-dir=node_modules \
  --exclude-dir=dist \
  && echo "⚠️  發現敏感資訊註解" || echo "✅ 未發現敏感註解"

echo "✅ 安全檢查完成"
```

#### Git Hooks

```bash
#!/bin/sh
# .git/hooks/pre-commit

echo "執行 pre-commit 檢查..."

# 執行安全掃描
./scripts/security-check.sh

if [ $? -ne 0 ]; then
  echo "❌ 提交被拒絕：發現安全問題"
  exit 1
fi

echo "✅ 安全檢查通過"
```

---

## 📐 8. 架構設計模式

### 前端架構模式

#### 組件組織原則

```
src/components/
├── common/           # 通用組件
│   ├── Button/
│   ├── Modal/
│   └── Card/
├── mobile/           # 手機優化組件
│   ├── MobileAddress/
│   ├── MobileDataCard/
│   └── MobileTabs/
├── features/         # 功能組件
│   ├── Hero/
│   │   ├── HeroCard.tsx
│   │   ├── HeroList.tsx
│   │   └── HeroMint.tsx
│   ├── Dungeon/
│   └── Altar/
└── layouts/          # 佈局組件
    ├── Header/
    ├── Sidebar/
    └── Footer/
```

#### 自定義 Hook 模式

```typescript
// 1. 數據獲取 Hook
export function useHeroData(tokenId: bigint) {
  return useQuery({
    queryKey: ['hero', tokenId],
    queryFn: () => fetchHeroFromSubgraph(tokenId),
    staleTime: 30_000,
    enabled: tokenId > 0n
  });
}

// 2. 合約互動 Hook
export function useHeroMint() {
  const { executeTransaction } = useContractTransaction();

  return useMutation({
    mutationFn: async (rarity: number) => {
      return executeTransaction({
        contractCall: {
          address: HERO_ADDRESS,
          abi: HeroABI,
          functionName: 'mintHero',
          args: [rarity]
        }
      });
    }
  });
}

// 3. 組合 Hook
export function useHeroManager(tokenId: bigint) {
  const heroData = useHeroData(tokenId);
  const mintHero = useHeroMint();
  const upgradeHero = useHeroUpgrade();

  return {
    hero: heroData.data,
    isLoading: heroData.isLoading,
    mint: mintHero.mutate,
    upgrade: upgradeHero.mutate
  };
}
```

#### 狀態管理模式 (Zustand)

```typescript
// stores/authStore.ts
interface AuthState {
  address: `0x${string}` | null;
  isConnected: boolean;
  vipLevel: number;
  connect: (address: `0x${string}`) => void;
  disconnect: () => void;
  setVipLevel: (level: number) => void;
}

export const useAuthStore = create<AuthState>((set) => ({
  address: null,
  isConnected: false,
  vipLevel: 0,

  connect: (address) => set({ address, isConnected: true }),
  disconnect: () => set({ address: null, isConnected: false, vipLevel: 0 }),
  setVipLevel: (level) => set({ vipLevel: level })
}));

// 使用
function Header() {
  const { address, isConnected, disconnect } = useAuthStore();

  if (!isConnected) return <ConnectButton />;

  return (
    <div>
      <span>{address}</span>
      <button onClick={disconnect}>Disconnect</button>
    </div>
  );
}
```

### 智能合約架構模式

#### Diamond Proxy 模式

```solidity
// DungeonCore.sol - Diamond 核心
contract DungeonCore {
    mapping(bytes4 => address) public facets;

    function addFacet(address facet, bytes4[] calldata selectors) external onlyOwner {
        for (uint i = 0; i < selectors.length; i++) {
            facets[selectors[i]] = facet;
        }
    }

    fallback() external payable {
        address facet = facets[msg.sig];
        require(facet != address(0), "Function does not exist");

        assembly {
            calldatacopy(0, 0, calldatasize())
            let result := delegatecall(gas(), facet, 0, calldatasize(), 0, 0)
            returndatacopy(0, 0, returndatasize())

            switch result
            case 0 { revert(0, returndatasize()) }
            default { return(0, returndatasize()) }
        }
    }
}
```

#### 存儲分離模式

```solidity
// DungeonStorage.sol - 數據存儲
contract DungeonStorage is Ownable {
    mapping(uint256 => Hero) public heroes;
    mapping(uint256 => Relic) public relics;
    mapping(uint256 => Party) public parties;

    mapping(address => bool) public authorizedContracts;

    modifier onlyAuthorized() {
        require(authorizedContracts[msg.sender], "Unauthorized");
        _;
    }

    function setHero(uint256 tokenId, Hero memory hero)
        external
        onlyAuthorized
    {
        heroes[tokenId] = hero;
    }
}

// Hero.sol - 業務邏輯
contract Hero is ERC721 {
    DungeonStorage public storageContract;

    function mintHero(uint8 rarity) external payable {
        uint256 tokenId = _mintToken(msg.sender);

        // 存儲到獨立合約
        storageContract.setHero(tokenId, Hero({
            rarity: rarity,
            power: calculatePower(rarity),
            level: 1
        }));
    }
}
```

#### 工廠模式

```solidity
// PartyFactory.sol
contract PartyFactory {
    event PartyCreated(uint256 indexed partyId, address indexed leader);

    mapping(uint256 => Party) public parties;
    uint256 public partyCount;

    function createParty(uint256[] calldata heroIds) external returns (uint256) {
        require(heroIds.length <= 5, "Max 5 heroes");

        uint256 partyId = ++partyCount;

        Party storage party = parties[partyId];
        party.leader = msg.sender;
        party.heroIds = heroIds;
        party.totalPower = calculateTotalPower(heroIds);
        party.createdAt = block.timestamp;

        emit PartyCreated(partyId, msg.sender);

        return partyId;
    }
}
```

### 子圖索引模式

#### Entity 關係設計

```graphql
# 一對多關係
type Player @entity {
  id: ID!
  heros: [Hero!]! @derivedFrom(field: "owner")
}

type Hero @entity {
  id: ID!
  owner: Player!
}

# 多對多關係
type Party @entity {
  id: ID!
  members: [PartyMember!]! @derivedFrom(field: "party")
}

type PartyMember @entity {
  id: ID!
  party: Party!
  hero: Hero!
  joinedAt: BigInt!
}

# 聚合數據
type DailyStats @entity {
  id: ID!                # 格式：YYYY-MM-DD
  totalExpeditions: BigInt!
  successfulExpeditions: BigInt!
  totalRewards: BigInt!
  uniquePlayers: BigInt!
}
```

#### 事件聚合模式

```typescript
// 更新玩家統計
export function updatePlayerStats(playerId: string, timestamp: BigInt): void {
  let player = Player.load(playerId);
  if (!player) {
    player = new Player(playerId);
    player.totalHeroes = BigInt.fromI32(0);
    player.totalExpeditions = BigInt.fromI32(0);
    player.registeredAt = timestamp;
  }

  // 更新活躍時間
  player.lastActiveAt = timestamp;

  // 計算總資產
  player.totalAssetValue = calculateAssetValue(player);

  player.save();
}

// 更新每日統計
export function updateDailyStats(date: string): void {
  let stats = DailyStats.load(date);
  if (!stats) {
    stats = new DailyStats(date);
    stats.totalExpeditions = BigInt.fromI32(0);
    stats.successfulExpeditions = BigInt.fromI32(0);
    stats.totalRewards = BigInt.fromI32(0);
    stats.uniquePlayers = BigInt.fromI32(0);
  }

  stats.totalExpeditions = stats.totalExpeditions.plus(BigInt.fromI32(1));
  stats.save();
}
```

---

## ⚡ 9. 性能優化策略

### 前端性能優化

#### 1. Code Splitting

```typescript
// 路由級別分割
import { lazy, Suspense } from 'react';

const HeroPage = lazy(() => import('./pages/HeroPage'));
const DungeonPage = lazy(() => import('./pages/DungeonPage'));
const AltarPage = lazy(() => import('./pages/AltarPage'));

function App() {
  return (
    <Suspense fallback={<LoadingSpinner />}>
      <Routes>
        <Route path="/hero" element={<HeroPage />} />
        <Route path="/dungeon" element={<DungeonPage />} />
        <Route path="/altar" element={<AltarPage />} />
      </Routes>
    </Suspense>
  );
}
```

#### 2. React 組件優化

```typescript
// 使用 React.memo 避免不必要的重新渲染
const HeroCard = React.memo(({ hero }: { hero: Hero }) => {
  return (
    <div className="hero-card">
      <img src={hero.image} alt={hero.name} />
      <h3>{hero.name}</h3>
      <p>Power: {hero.power}</p>
    </div>
  );
}, (prevProps, nextProps) => {
  // 自定義比較邏輯
  return prevProps.hero.tokenId === nextProps.hero.tokenId &&
         prevProps.hero.power === nextProps.hero.power;
});

// 使用 useMemo 緩存計算結果
function HeroList({ heroes }: { heroes: Hero[] }) {
  const sortedHeroes = useMemo(() => {
    return [...heroes].sort((a, b) => Number(b.power - a.power));
  }, [heroes]);

  const totalPower = useMemo(() => {
    return heroes.reduce((sum, hero) => sum + Number(hero.power), 0);
  }, [heroes]);

  return <div>{/* 渲染邏輯 */}</div>;
}

// 使用 useCallback 穩定函數引用
function HeroManager() {
  const [selectedHero, setSelectedHero] = useState<Hero | null>(null);

  const handleSelect = useCallback((hero: Hero) => {
    setSelectedHero(hero);
  }, []);

  return <HeroList heroes={heroes} onSelect={handleSelect} />;
}
```

#### 3. 圖片優化

```typescript
// 懶加載圖片
function LazyImage({ src, alt }: { src: string; alt: string }) {
  const [isLoaded, setIsLoaded] = useState(false);
  const imgRef = useRef<HTMLImageElement>(null);

  useEffect(() => {
    if (!imgRef.current) return;

    const observer = new IntersectionObserver(
      ([entry]) => {
        if (entry.isIntersecting) {
          setIsLoaded(true);
          observer.disconnect();
        }
      },
      { rootMargin: '50px' }
    );

    observer.observe(imgRef.current);

    return () => observer.disconnect();
  }, []);

  return (
    <img
      ref={imgRef}
      src={isLoaded ? src : '/placeholder.png'}
      alt={alt}
      loading="lazy"
    />
  );
}
```

#### 4. 虛擬滾動

```typescript
import { useVirtualizer } from '@tanstack/react-virtual';

function HeroListVirtual({ heroes }: { heroes: Hero[] }) {
  const parentRef = useRef<HTMLDivElement>(null);

  const virtualizer = useVirtualizer({
    count: heroes.length,
    getScrollElement: () => parentRef.current,
    estimateSize: () => 100,  // 每個項目的預估高度
    overscan: 5  // 預渲染項目數
  });

  return (
    <div ref={parentRef} style={{ height: '600px', overflow: 'auto' }}>
      <div
        style={{
          height: `${virtualizer.getTotalSize()}px`,
          width: '100%',
          position: 'relative'
        }}
      >
        {virtualizer.getVirtualItems().map((item) => (
          <div
            key={item.key}
            style={{
              position: 'absolute',
              top: 0,
              left: 0,
              width: '100%',
              height: `${item.size}px`,
              transform: `translateY(${item.start}px)`
            }}
          >
            <HeroCard hero={heroes[item.index]} />
          </div>
        ))}
      </div>
    </div>
  );
}
```

#### 5. TanStack Query 優化

```typescript
// 預取數據
function HeroListPage() {
  const queryClient = useQueryClient();

  // 預取下一頁
  const prefetchNextPage = useCallback(async (nextPage: number) => {
    await queryClient.prefetchQuery({
      queryKey: ['heroes', nextPage],
      queryFn: () => fetchHeroes(nextPage)
    });
  }, [queryClient]);

  return <HeroList onPageChange={prefetchNextPage} />;
}

// 樂觀更新
const { mutate } = useMutation({
  mutationFn: upgradeHero,
  onMutate: async (heroId) => {
    // 取消正在進行的查詢
    await queryClient.cancelQueries({ queryKey: ['hero', heroId] });

    // 快照舊數據
    const previousHero = queryClient.getQueryData(['hero', heroId]);

    // 樂觀更新
    queryClient.setQueryData(['hero', heroId], (old: Hero) => ({
      ...old,
      level: old.level + 1
    }));

    return { previousHero };
  },
  onError: (err, heroId, context) => {
    // 回滾
    queryClient.setQueryData(['hero', heroId], context?.previousHero);
  },
  onSettled: (heroId) => {
    // 重新驗證
    queryClient.invalidateQueries({ queryKey: ['hero', heroId] });
  }
});
```

### 子圖性能優化

#### 1. 查詢優化

```graphql
# ❌ 低效查詢：獲取所有字段
query GetHeroes {
  heroes {
    id
    tokenId
    owner { id }
    rarity
    power
    level
    experience
    mintedAt
    lastUpgradedAt
    transferHistory {
      from
      to
      timestamp
    }
  }
}

# ✅ 優化查詢：只獲取需要的字段
query GetHeroes {
  heroes {
    id
    tokenId
    rarity
    power
  }
}
```

#### 2. 分頁策略

```typescript
// 游標分頁
async function fetchHeroesWithCursor(cursor?: string, limit = 50) {
  const query = `
    query GetHeroes($cursor: String, $limit: Int!) {
      heroes(
        first: $limit
        where: { id_gt: $cursor }
        orderBy: id
        orderDirection: asc
      ) {
        id
        tokenId
        rarity
        power
      }
    }
  `;

  const result = await fetchSubgraph(query, { cursor, limit });

  return {
    heroes: result.heroes,
    nextCursor: result.heroes[result.heroes.length - 1]?.id
  };
}
```

#### 3. 批次查詢

```typescript
// 批次獲取多個 Hero
async function fetchHeroesBatch(tokenIds: string[]) {
  const query = `
    query GetHeroesBatch($ids: [ID!]!) {
      heroes(where: { id_in: $ids }) {
        id
        tokenId
        rarity
        power
      }
    }
  `;

  return fetchSubgraph(query, { ids: tokenIds });
}
```

### 智能合約 Gas 優化

#### 1. Storage 優化

```solidity
// ❌ 低效：多次存儲讀寫
function upgradeHero(uint256 tokenId) external {
    heroes[tokenId].level += 1;
    heroes[tokenId].power += 100;
    heroes[tokenId].lastUpgraded = block.timestamp;
}

// ✅ 優化：緩存到內存
function upgradeHero(uint256 tokenId) external {
    Hero memory hero = heroes[tokenId];
    hero.level += 1;
    hero.power += 100;
    hero.lastUpgraded = block.timestamp;
    heroes[tokenId] = hero;
}
```

#### 2. 變數打包

```solidity
// ❌ 低效：每個變數佔用一個 slot (32 bytes)
struct Hero {
    uint256 power;      // slot 0
    uint256 level;      // slot 1
    uint8 rarity;       // slot 2
    bool isAscended;    // slot 3
}

// ✅ 優化：打包到更少的 slot
struct Hero {
    uint128 power;      // slot 0 (16 bytes)
    uint64 level;       // slot 0 (8 bytes)
    uint8 rarity;       // slot 0 (1 byte)
    bool isAscended;    // slot 0 (1 byte)
    // 共用 slot 0，節省 3 個 slot
}
```

#### 3. Calldata vs Memory

```solidity
// ❌ 低效：複製到 memory
function batchMint(uint8[] memory rarities) external {
    for (uint i = 0; i < rarities.length; i++) {
        _mintHero(rarities[i]);
    }
}

// ✅ 優化：直接使用 calldata
function batchMint(uint8[] calldata rarities) external {
    for (uint i = 0; i < rarities.length; i++) {
        _mintHero(rarities[i]);
    }
}
```

#### 4. 事件優化

```solidity
// ❌ 低效：未索引的事件
event HeroMinted(uint256 tokenId, address owner, uint8 rarity);

// ✅ 優化：索引關鍵字段（最多 3 個）
event HeroMinted(
    uint256 indexed tokenId,
    address indexed owner,
    uint8 rarity  // 不索引小數值
);
```

### 後端性能優化

#### 1. Redis 快取策略

```javascript
// 分層快取
class CacheService {
  constructor() {
    // L1: 內存快取（5分鐘）
    this.memCache = new NodeCache({ stdTTL: 300, checkperiod: 60 });

    // L2: Redis 快取（1小時）
    this.redis = new Redis(process.env.REDIS_URL);
  }

  async get(key) {
    // 檢查 L1
    let value = this.memCache.get(key);
    if (value) return value;

    // 檢查 L2
    const redisValue = await this.redis.get(key);
    if (redisValue) {
      value = JSON.parse(redisValue);
      this.memCache.set(key, value);
      return value;
    }

    return null;
  }

  async set(key, value, ttl = 3600) {
    this.memCache.set(key, value, Math.min(ttl, 300));
    await this.redis.setex(key, ttl, JSON.stringify(value));
  }

  // 批次設置
  async mset(entries) {
    const pipeline = this.redis.pipeline();

    for (const [key, value, ttl] of entries) {
      this.memCache.set(key, value, Math.min(ttl || 300, 300));
      pipeline.setex(key, ttl || 3600, JSON.stringify(value));
    }

    await pipeline.exec();
  }
}
```

#### 2. 連接池優化

```javascript
// PostgreSQL 連接池
const { Pool } = require('pg');

const pool = new Pool({
  host: process.env.DB_HOST,
  port: process.env.DB_PORT,
  database: process.env.DB_NAME,
  user: process.env.DB_USER,
  password: process.env.DB_PASSWORD,
  max: 20,              // 最大連接數
  idleTimeoutMillis: 30000,
  connectionTimeoutMillis: 2000,
});

// 使用連接池
async function queryDatabase(sql, params) {
  const client = await pool.connect();
  try {
    const result = await client.query(sql, params);
    return result.rows;
  } finally {
    client.release();
  }
}
```

#### 3. 響應壓縮

```javascript
const compression = require('compression');

app.use(compression({
  level: 6,           // 壓縮級別 (0-9)
  threshold: 1024,    // 只壓縮大於 1KB 的響應
  filter: (req, res) => {
    // 自定義過濾器
    if (req.headers['x-no-compression']) {
      return false;
    }
    return compression.filter(req, res);
  }
}));
```

---

## 📊 10. 監控指標

### 前端性能指標

#### Core Web Vitals

```typescript
// 測量性能指標
import { onLCP, onFID, onCLS, onFCP, onTTFB } from 'web-vitals';

function reportWebVitals() {
  onLCP((metric) => {
    console.log('Largest Contentful Paint:', metric.value);
    // 目標: < 2.5s
  });

  onFID((metric) => {
    console.log('First Input Delay:', metric.value);
    // 目標: < 100ms
  });

  onCLS((metric) => {
    console.log('Cumulative Layout Shift:', metric.value);
    // 目標: < 0.1
  });

  onFCP((metric) => {
    console.log('First Contentful Paint:', metric.value);
    // 目標: < 1.8s
  });

  onTTFB((metric) => {
    console.log('Time to First Byte:', metric.value);
    // 目標: < 600ms
  });
}
```

#### 自定義指標

```typescript
// 測量 React 組件渲染時間
function measureComponentRender(componentName: string) {
  performance.mark(`${componentName}-start`);

  useEffect(() => {
    performance.mark(`${componentName}-end`);
    performance.measure(
      componentName,
      `${componentName}-start`,
      `${componentName}-end`
    );

    const measure = performance.getEntriesByName(componentName)[0];
    console.log(`${componentName} render time:`, measure.duration);

    // 清理
    performance.clearMarks();
    performance.clearMeasures();
  });
}

// 測量 API 請求時間
async function fetchWithMetrics(url: string) {
  const startTime = performance.now();

  try {
    const response = await fetch(url);
    const data = await response.json();

    const endTime = performance.now();
    const duration = endTime - startTime;

    console.log(`API ${url} took ${duration}ms`);

    return data;
  } catch (error) {
    console.error(`API ${url} failed`);
    throw error;
  }
}
```

### 智能合約指標

#### Gas 使用報告

```bash
# Foundry Gas Report
forge test --gas-report

# 輸出範例
| Contract      | Function     | Avg Gas | Max Gas |
|---------------|--------------|---------|---------|
| Hero          | mintHero     | 125,431 | 145,678 |
| Hero          | upgradeHero  | 67,234  | 89,456  |
| DungeonMaster | startExpedition | 234,567 | 287,345 |
```

#### 鏈上交易監控

```typescript
// 監控合約事件
const heroContract = new ethers.Contract(HERO_ADDRESS, HeroABI, provider);

heroContract.on('HeroMinted', (tokenId, owner, rarity, event) => {
  console.log('New Hero Minted:', {
    tokenId: tokenId.toString(),
    owner,
    rarity,
    txHash: event.transactionHash,
    gasUsed: event.gasUsed?.toString(),
    blockNumber: event.blockNumber
  });
});
```

### 子圖健康監控

#### 同步狀態檢查

```typescript
async function checkSubgraphHealth() {
  const query = `
    {
      _meta {
        block {
          number
          timestamp
        }
        deployment
        hasIndexingErrors
      }
    }
  `;

  const result = await fetchSubgraph(query);
  const meta = result._meta;

  const currentBlock = await provider.getBlockNumber();
  const syncLag = currentBlock - meta.block.number;

  console.log('Subgraph Health:', {
    indexedBlock: meta.block.number,
    currentBlock,
    syncLag,
    hasErrors: meta.hasIndexingErrors
  });

  // 告警閾值
  if (syncLag > 100) {
    console.error('⚠️ Subgraph sync lag too high:', syncLag);
  }

  if (meta.hasIndexingErrors) {
    console.error('❌ Subgraph has indexing errors');
  }
}
```

### 後端 API 監控

#### 響應時間追蹤

```javascript
// 中間件
app.use((req, res, next) => {
  const start = Date.now();

  res.on('finish', () => {
    const duration = Date.now() - start;

    logger.info('API Request', {
      method: req.method,
      url: req.url,
      statusCode: res.statusCode,
      duration: `${duration}ms`,
      userAgent: req.get('user-agent')
    });

    // 慢查詢告警
    if (duration > 1000) {
      logger.warn('Slow API Request', {
        url: req.url,
        duration: `${duration}ms`
      });
    }
  });

  next();
});
```

#### 錯誤率監控

```javascript
let errorCount = 0;
let totalRequests = 0;

app.use((err, req, res, next) => {
  errorCount++;
  totalRequests++;

  const errorRate = (errorCount / totalRequests) * 100;

  logger.error('API Error', {
    error: err.message,
    stack: err.stack,
    url: req.url,
    method: req.method,
    errorRate: `${errorRate.toFixed(2)}%`
  });

  // 錯誤率告警
  if (errorRate > 5) {
    logger.error('⚠️ High error rate detected:', errorRate);
  }

  res.status(500).json({ error: 'Internal Server Error' });
});
```

### 關鍵指標儀表板

#### 前端指標

```yaml
性能指標:
  - FCP (First Contentful Paint): < 1.5s
  - LCP (Largest Contentful Paint): < 2.5s
  - FID (First Input Delay): < 100ms
  - CLS (Cumulative Layout Shift): < 0.1
  - TTI (Time to Interactive): < 3s

資源指標:
  - Bundle 大小 (gzipped): < 500KB
  - 初始 JS 加載: < 200KB
  - 圖片優化率: > 80%
  - 快取命中率: > 70%

使用者體驗:
  - 頁面載入時間: < 2s
  - API 響應時間: < 500ms
  - 錯誤率: < 1%
```

#### 合約指標

```yaml
Gas 效率:
  - Hero 鑄造: < 150,000 gas
  - Relic 鑄造: < 120,000 gas
  - 探險執行: < 250,000 gas
  - Hero 升階: < 100,000 gas

交易成功率:
  - 整體成功率: > 99%
  - VRF 回調成功率: > 98%
  - Revert 率: < 1%

合約安全:
  - 緊急暫停功能: 可用
  - 權限檢查: 100%
  - 重入保護: 全覆蓋
```

#### 子圖指標

```yaml
同步狀態:
  - 區塊同步延遲: < 10 blocks
  - 同步成功率: > 99.9%
  - 索引錯誤率: < 0.1%

查詢性能:
  - 平均查詢時間: < 300ms
  - P95 查詢時間: < 500ms
  - P99 查詢時間: < 1s
  - 查詢錯誤率: < 0.5%

資料完整性:
  - 事件索引率: 100%
  - 數據一致性: 100%
```

#### 後端指標

```yaml
API 性能:
  - 平均響應時間: < 150ms
  - P95 響應時間: < 200ms
  - P99 響應時間: < 500ms
  - 吞吐量: > 1000 req/min

快取效率:
  - Redis 命中率: > 85%
  - 內存快取命中率: > 90%
  - 快取過期率: < 5%

穩定性:
  - 服務可用性: > 99.9%
  - 錯誤率: < 0.5%
  - 記憶體使用: < 450MB
  - CPU 使用: < 70%
```

---

## 🎓 附錄

### A. 開發環境設置

#### 必要工具安裝

```bash
# 1. Node.js (>= 18)
brew install node@18

# 2. Git
brew install git

# 3. Foundry
curl -L https://foundry.paradigm.xyz | bash
foundryup

# 4. Graph CLI
npm install -g @graphprotocol/graph-cli

# 5. Vercel CLI
npm install -g vercel

# 6. 程式碼編輯器
# 推薦：Visual Studio Code + Solidity 插件
```

#### 專案初始化

```bash
# 1. 克隆所有倉庫
cd ~/Documents
git clone https://github.com/your-org/DungeonDelversContracts.git
git clone https://github.com/your-org/SoulboundSaga.git
git clone https://github.com/your-org/dungeon-delvers-subgraph.git
git clone https://github.com/your-org/dungeon-delvers-metadata-server.git

# 2. 安裝依賴
cd DungeonDelversContracts && npm install
cd ../SoulboundSaga && npm install
cd ../dungeon-delvers-subgraph && npm install
cd ../dungeon-delvers-metadata-server && npm install

# 3. 配置環境變數
# 複製 .env.example 到 .env 並填入配置
cp .env.example .env

# 4. 啟動本地開發
# 每個專案在獨立終端窗口啟動
```

### B. 常用命令速查

#### 智能合約

```bash
# 編譯
npm run compile

# 測試
npm run test
forge test

# 部署（測試網）
npm run deploy:testnet

# 部署（主網）
npm run deploy

# 驗證合約
npm run verify

# Gas 報告
forge test --gas-report
```

#### 前端

```bash
# 開發模式
npm run dev

# 類型檢查
npm run type-check

# Lint 檢查
npm run lint

# 測試
npm run test

# 構建
npm run build

# 預覽
npm run preview

# 部署
vercel --prod
```

#### 子圖

```bash
# 代碼生成
npm run codegen

# 構建
npm run build

# 部署到 Goldsky
npm run deploy:goldsky

# 部署到 Studio
npm run deploy

# 測試
npm run test
```

#### 後端

```bash
# 開發模式
npm run dev

# 生產模式
npm run start

# 測試
npm run test

# Lint
npm run lint
```

### C. 故障排查指南

#### 常見問題

**1. 合約部署失敗**
```bash
# 檢查 Gas Price
# 檢查賬戶餘額
# 驗證 RPC 連接
# 檢查合約代碼語法
```

**2. 子圖同步延遲**
```bash
# 檢查子圖健康狀態
# 驗證 RPC 端點
# 檢查事件監聽配置
# 查看 Goldsky 日誌
```

**3. 前端連接問題**
```bash
# 清除瀏覽器快取
# 檢查錢包連接
# 驗證 RPC 配置
# 檢查環境變數
```

**4. API 響應緩慢**
```bash
# 檢查 Redis 連接
# 查看快取命中率
# 分析慢查詢日誌
# 驗證資料庫連接池
```

### D. 版本發布流程

```bash
# 1. 更新版本號
npm version patch/minor/major

# 2. 更新 CHANGELOG.md
# 記錄本次更新內容

# 3. 提交變更
git add .
git commit -m "chore: release v2.0.1.0"

# 4. 創建標籤
git tag v2.0.1.0

# 5. 推送到遠端
git push origin main --tags

# 6. 部署各個服務
npm run deploy  # 各專案
```

---

## 📚 參考資源

### 官方文檔
- [Base Network](https://docs.base.org/)
- [The Graph](https://thegraph.com/docs/)
- [Chainlink VRF](https://docs.chain.link/vrf)
- [OpenZeppelin Contracts](https://docs.openzeppelin.com/contracts/)
- [Wagmi](https://wagmi.sh/)
- [Viem](https://viem.sh/)

### 開發工具
- [Hardhat](https://hardhat.org/docs)
- [Foundry Book](https://book.getfoundry.sh/)
- [Vite](https://vitejs.dev/)
- [TanStack Query](https://tanstack.com/query/)
- [Zustand](https://zustand-demo.pmnd.rs/)

### 社區資源
- [Ethereum Stack Exchange](https://ethereum.stackexchange.com/)
- [Base Discord](https://discord.gg/base)
- [The Graph Discord](https://discord.gg/graphprotocol)

---

**文檔維護者**: DungeonDelvers Core Team
**聯繫方式**: dev@dungeondelvers.io
**最後更新**: 2025-01-12

---

*本文檔隨專案持續更新，請定期查看最新版本。*
