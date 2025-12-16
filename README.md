# TranslateX - 国际化管理平台

> 基于 Turborepo + NestJS + Vue 3 的企业级国际化解决方案

## 文档导航

| 文档 | 描述 |
|------|------|
| [i18n-platform-design.md](./i18n-platform-design.md) | 核心设计文档（系统架构、数据库、API设计）|
| [project-structure.md](./project-structure.md) | Turborepo 项目结构说明 |
| [workflows-diagrams.md](./workflows-diagrams.md) | 10+ Mermaid 流程图（CLI/AI/版本管理流程）|
| [implementation-guide.md](./implementation-guide.md) | 实现指南（代码示例、开发计划）|
| [CHANGELOG.md](./CHANGELOG.md) | 更新日志 |

## 核心功能

### CLI 工具
```bash
i18n-cli init                          # 初始化项目
i18n-cli extract                       # 提取词条（支持 Vue/React）
i18n-cli push                          # 上传到平台
i18n-cli pull --locales en-US,ja-JP    # 下载翻译
i18n-cli version:create v1.0.0         # 创建版本
i18n-cli version:master v1.0.0         # 标记主版本
```

### 管理平台
- 📝 词条管理（批量导入/导出）
- 🤖 AI 自动翻译（OpenAI/Google Translate）
- 👥 多人协作翻译
- 📊 翻译进度跟踪
- 🔄 版本管理（与 Git 分支对应）
- ✔️ 翻译审核流程

## 技术栈

```
┌────────────────────────────────────────┐
│        Turborepo Monorepo           │
├────────────────────────────────────────┤
│ Apps:                               │
│  - web (Vue 3 + TypeScript)         │
│  - api (NestJS + Sequelize)         │
│                                     │
│ Packages:                           │
│  - cli (Commander.js)               │
│  - parser (AST 解析)                │
│  - sdk (API Client)                 │
│  - shared (共享类型)                │
└────────────────────────────────────────┘
```

- **Monorepo**: Turborepo + pnpm
- **后端**: NestJS + Sequelize + MySQL + Redis
- **前端**: Vue 3 + TypeScript + Element Plus
- **CLI**: Node.js + Commander + Babel/Vue Parser
- **AI**: OpenAI API

## 快速开始

```bash
# 安装依赖
pnpm install

# 配置环境变量
cp apps/api/.env.example apps/api/.env
cp apps/web/.env.example apps/web/.env

# 数据库迁移
pnpm --filter @translatex/api db:migrate

# 启动开发服务
pnpm dev

# Web:  http://localhost:5173
# API:  http://localhost:3000
```

## 包清单

| 包名 | 描述 | 主要依赖 |
|------|------|---------|
| `@translatex/web` | 前端管理平台 | Vue 3, Element Plus, Pinia |
| `@translatex/api` | 后端 API 服务 | NestJS, Sequelize, Bull |
| `@translatex/cli` | 命令行工具 | Commander, Inquirer, Ora |
| `@translatex/parser` | 词条解析器 | @babel/parser, @vue/compiler-sfc |
| `@translatex/sdk` | API 客户端 | Axios |
| `@translatex/shared` | 共享类型和工具 | - |

## 开发里程碑

- **Week 1-2**: 基础设施 + 共享包
- **Week 3-4**: 解析器 + CLI 工具
- **Week 4-6**: 后端 API + AI 翻译
- **Week 6-8**: 前端管理平台
- **Week 9-10**: 测试 + 优化 + 上线

## License

MIT
