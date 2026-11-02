# SelAi 技术选型

## 技术原则

1. **本地优先（Local-first）** - 所有数据存储在用户设备上，不依赖云端
2. **隐私安全（Privacy & Security）** - 敏感数据加密，可选离线 AI 推理
3. **简单可维护（Simplicity）** - 技术栈不过度复杂，便于个人开发者维护
4. **可扩展（Extensible）** - 模块化设计，方便后续增加功能
5. **跨平台（Cross-platform）** - 支持 macOS、Windows、Linux，未来扩展到移动端

## 技术栈选择

### 桌面应用框架

**推荐：Tauri 2.0**

| 方案 | 优势 | 劣势 | 结论 |
|------|------|------|------|
| Electron | 生态成熟，开发快 | 包体积大（100MB+），内存占用高 | ❌ 不适合本地优先应用 |
| Tauri 2.0 | 包体积小（10MB），性能好，Rust 后端安全 | 生态较新，部分组件不成熟 | ✅ **推荐** |
| 原生应用 | 性能最优 | 开发成本高，需维护多平台 | ❌ 不适合 MVP |

**选择 Tauri 的理由：**
- 包体积小，符合「轻量级工具」定位
- Rust 后端性能好，适合处理本地文件 I/O
- 前端用 Web 技术，开发效率高
- 内置安全机制，适合处理敏感数据

### 前端技术栈

**推荐：React 18 + TypeScript + Vite**

| 方案 | 优势 | 劣势 |
|------|------|------|
| React | 生态最大，组件库丰富 | 学习曲线平缓但精通难 |
| Vue | 上手快，文档好 | 生态不如 React 丰富 |
| Svelte | 性能好，代码简洁 | 生态较小 |

**选择 React 的理由：**
- 生态最丰富，需要的组件基本都有现成的
- TypeScript 支持好，适合复杂应用
- 未来扩展到移动端可以用 React Native 复用代码

**前端关键依赖：**
- UI 框架：Tailwind CSS + shadcn/ui（轻量、可定制）
- 状态管理：Zustand（简单、TypeScript 友好）
- 图表：Recharts（React 原生，简单易用）
- Markdown 渲染：react-markdown
- 代码编辑器（日志编辑）：CodeMirror 6 或 Monaco Editor

### 数据存储

**推荐：本地文件系统 + SQLite**

```
data/
├── logs/              # 原始日志文件（Markdown 格式）
│   ├── 2026/
│   │   ├── 11/
│   │   │   ├── 2026-11-03.md
│   │   │   ├── 2026-11-04.md
│   │   │   └── ...
│   │   └── ...
│   └── ...
├── index.db           # SQLite 索引数据库
├── config.json        # 用户配置
└── exports/           # 导出数据
```

**为什么不用纯数据库？**
- Markdown 文件人类可读可编辑
- 版本控制友好（可以用 Git 管理）
- 不依赖特定软件
- AI 模型容易理解 Markdown 格式

**SQLite 的作用：**
- 全文搜索索引
- 标签/分类查询
- 统计分析（聚合查询）
- 时间序列数据

### AI 接入

**分层策略：**

#### Layer 1: 规则引擎（无 AI 成本）
- 简单的数据提取（正则匹配）
- 基础统计（求和、平均、趋势）
- 标签分类（关键词匹配）

#### Layer 2: 本地模型（隐私优先）
- 工具：Ollama / LM Studio
- 模型：Llama 3, Qwen 2.5, Mistral
- 用途：敏感数据的初步分析、离线场景

#### Layer 3: 云端模型（能力优先）
- 工具：Claude API / OpenAI API / 智谱 API
- 用途：深度分析、报告生成、决策辅助
- 注意：敏感数据可选不上传

**多模型架构：**
```typescript
interface AIProvider {
  name: string;
  analyze(prompt: string, context: LogContext): Promise<Analysis>;
  generateReport(data: AnalysisData, template: ReportTemplate): Promise<string>;
  advise(decision: DecisionContext): Promise<Advice>;
}

class AIEngine {
  private providers: AIProvider[];
  
  // 重要决策可以并行咨询多个模型
  async crossValidate(decision: DecisionContext): Promise<MultiModelAdvice> {
    const results = await Promise.all(
      this.providers.map(p => p.advise(decision))
    );
    return this.mergeAdvice(results);
  }
}
```

### 数据同步（Phase 2）

**推荐：端到端加密 + 可选云同步**

- 本地优先，不强制云同步
- 可选方案：
  - iCloud / Dropbox（简单，但安全性依赖平台）
  - 自建 WebDAV（用户掌控）
  - 端到端加密同步（最安全，实现复杂）

## 系统架构

```
┌─────────────────────────────────────────────────────┐
│                    Tauri App                         │
│  ┌───────────────────────────────────────────────┐  │
│  │              React Frontend                    │  │
│  │  ┌─────────┐ ┌──────────┐ ┌───────────────┐  │  │
│  │  │ Logger  │ │Dashboard │ │ Decision Aid  │  │  │
│  │  └─────────┘ └──────────┘ └───────────────┘  │  │
│  └───────────────────────────────────────────────┘  │
│                        ↕ IPC                         │
│  ┌───────────────────────────────────────────────┐  │
│  │              Rust Backend                      │  │
│  │  ┌─────────┐ ┌──────────┐ ┌───────────────┐  │  │
│  │  │ File I/O│ │ SQLite   │ │ AI Engine     │  │  │
│  │  └─────────┘ └──────────┘ └───────────────┘  │  │
│  └───────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────┘
         ↕                    ↕                  ↕
   Local Files           SQLite DB        AI Providers
   (Markdown)            (Index)          (Local/Cloud)
```

## 开发环境搭建

```bash
# 前置要求
# - Node.js 18+
# - Rust (for Tauri)
# - pnpm or yarn

# 安装依赖
pnpm install

# 启动开发环境
pnpm tauri dev

# 构建生产版本
pnpm tauri build
```

## 技术风险与缓解

| 风险 | 影响 | 概率 | 缓解措施 |
|------|------|------|----------|
| Tauri 生态不成熟 | 中 | 中 | 提前验证关键功能，必要时回退 Electron |
| 本地 AI 模型能力不足 | 高 | 高 | 分层策略，关键功能用云端模型 |
| 数据量增长后性能下降 | 中 | 低 | SQLite 索引优化，分片存储 |
| 跨平台兼容性问题 | 中 | 中 | 尽早进行多平台测试 |

## 后续技术决策点

以下问题需要在开发过程中进一步评估：

1. **是否引入向量数据库？** - 用于语义搜索和相似经历检索
2. **移动端用什么技术？** - React Native（复用代码）vs Flutter（性能更好）
3. **是否需要后端服务？** - 如果做多设备同步，可能需要轻量后端
4. **AI 模型微调？** - 是否用用户数据微调专属模型（隐私风险需评估）
