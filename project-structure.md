# Turborepo 项目结构详细说明

## 完整目录结构

```
i18n-platform/
├── .github/
│   └── workflows/
│       ├── ci.yml                    # CI/CD 配置
│       └── release.yml               # 发布流程
│
├── apps/
│   ├── web/                          # 前端管理平台
│   │   ├── public/
│   │   ├── src/
│   │   │   ├── assets/              # 静态资源
│   │   │   ├── components/          # 公共组件
│   │   │   │   ├── EntryTable/
│   │   │   │   ├── TranslationEditor/
│   │   │   │   └── VersionSelector/
│   │   │   ├── views/               # 页面视图
│   │   │   │   ├── Dashboard.vue
│   │   │   │   ├── Projects/
│   │   │   │   │   ├── ProjectList.vue
│   │   │   │   │   ├── ProjectDetail.vue
│   │   │   │   │   └── ProjectSettings.vue
│   │   │   │   ├── Entries/
│   │   │   │   │   ├── EntryList.vue
│   │   │   │   │   └── EntryEditor.vue
│   │   │   │   ├── Translations/
│   │   │   │   │   ├── TranslationBoard.vue
│   │   │   │   │   └── BatchTranslate.vue
│   │   │   │   └── Versions/
│   │   │   │       ├── VersionList.vue
│   │   │   │       └── VersionCompare.vue
│   │   │   ├── router/              # 路由配置
│   │   │   ├── store/               # Pinia Store
│   │   │   │   ├── project.ts
│   │   │   │   ├── entry.ts
│   │   │   │   └── translation.ts
│   │   │   ├── api/                 # API 调用
│   │   │   │   ├── client.ts
│   │   │   │   ├── project.ts
│   │   │   │   ├── entry.ts
│   │   │   │   └── translation.ts
│   │   │   ├── utils/
│   │   │   ├── App.vue
│   │   │   └── main.ts
│   │   ├── index.html
│   │   ├── vite.config.ts
│   │   ├── package.json
│   │   └── tsconfig.json
│   │
│   └── api/                          # 后端 API 服务
│       ├── src/
│       │   ├── common/              # 公共模块
│       │   │   ├── decorators/
│       │   │   ├── filters/
│       │   │   ├── guards/
│       │   │   │   └── api-key.guard.ts
│       │   │   ├── interceptors/
│       │   │   └── pipes/
│       │   ├── config/              # 配置
│       │   │   ├── database.config.ts
│       │   │   ├── redis.config.ts
│       │   │   └── ai.config.ts
│       │   ├── modules/
│       │   │   ├── auth/            # 认证模块
│       │   │   │   ├── auth.controller.ts
│       │   │   │   ├── auth.service.ts
│       │   │   │   ├── auth.module.ts
│       │   │   │   └── strategies/
│       │   │   │       ├── jwt.strategy.ts
│       │   │   │       └── api-key.strategy.ts
│       │   │   ├── user/            # 用户模块
│       │   │   │   ├── user.controller.ts
│       │   │   │   ├── user.service.ts
│       │   │   │   ├── user.module.ts
│       │   │   │   ├── entities/
│       │   │   │   │   └── user.entity.ts
│       │   │   │   └── dto/
│       │   │   ├── project/         # 项目模块
│       │   │   │   ├── project.controller.ts
│       │   │   │   ├── project.service.ts
│       │   │   │   ├── project.module.ts
│       │   │   │   ├── entities/
│       │   │   │   │   └── project.entity.ts
│       │   │   │   └── dto/
│       │   │   │       ├── create-project.dto.ts
│       │   │   │       └── update-project.dto.ts
│       │   │   ├── entry/           # 词条模块
│       │   │   │   ├── entry.controller.ts
│       │   │   │   ├── entry.service.ts
│       │   │   │   ├── entry.module.ts
│       │   │   │   ├── entities/
│       │   │   │   │   └── entry.entity.ts
│       │   │   │   └── dto/
│       │   │   │       ├── batch-create-entries.dto.ts
│       │   │   │       └── find-entries.dto.ts
│       │   │   ├── translation/     # 翻译模块
│       │   │   │   ├── translation.controller.ts
│       │   │   │   ├── translation.service.ts
│       │   │   │   ├── translation.module.ts
│       │   │   │   ├── entities/
│       │   │   │   │   ├── translation.entity.ts
│       │   │   │   │   └── translation-job.entity.ts
│       │   │   │   └── dto/
│       │   │   ├── version/         # 版本模块
│       │   │   │   ├── version.controller.ts
│       │   │   │   ├── version.service.ts
│       │   │   │   ├── version.module.ts
│       │   │   │   ├── entities/
│       │   │   │   │   ├── version.entity.ts
│       │   │   │   │   └── entry-version.entity.ts
│       │   │   │   └── dto/
│       │   │   ├── ai/              # AI 翻译模块
│       │   │   │   ├── ai.controller.ts
│       │   │   │   ├── ai.service.ts
│       │   │   │   ├── ai.module.ts
│       │   │   │   └── providers/
│       │   │   │       ├── openai.service.ts
│       │   │   │       ├── google-translate.service.ts
│       │   │   │       └── deepl.service.ts
│       │   │   ├── queue/           # 队列模块
│       │   │   │   ├── queue.module.ts
│       │   │   │   ├── processors/
│       │   │   │   │   └── translation.processor.ts
│       │   │   │   └── producers/
│       │   │   └── export/          # 导出模块
│       │   │       ├── export.controller.ts
│       │   │       ├── export.service.ts
│       │   │       └── export.module.ts
│       │   ├── database/
│       │   │   └── migrations/      # 数据库迁移
│       │   ├── app.module.ts
│       │   ├── main.ts
│       │   └── worker.ts            # Queue Worker 入口
│       ├── test/
│       ├── package.json
│       ├── tsconfig.json
│       └── nest-cli.json
│
├── packages/
│   ├── cli/                          # CLI 工具包
│   │   ├── bin/
│   │   │   └── i18n-cli.js          # 可执行文件
│   │   ├── src/
│   │   │   ├── commands/
│   │   │   │   ├── init.ts          # 初始化命令
│   │   │   │   ├── extract.ts       # 提取命令
│   │   │   │   ├── push.ts          # 上传命令
│   │   │   │   ├── pull.ts          # 下载命令
│   │   │   │   ├── status.ts        # 状态命令
│   │   │   │   └── version/
│   │   │   │       ├── create.ts
│   │   │   │       └── master.ts
│   │   │   ├── config/
│   │   │   │   ├── loader.ts        # 配置加载器
│   │   │   │   └── schema.ts        # 配置验证
│   │   │   ├── utils/
│   │   │   │   ├── logger.ts
│   │   │   │   ├── file-scanner.ts
│   │   │   │   └── spinner.ts
│   │   │   ├── index.ts
│   │   │   └── cli.ts
│   │   ├── templates/               # 配置模板
│   │   │   └── i18n.config.template.js
│   │   ├── package.json
│   │   └── tsconfig.json
│   │
│   ├── parser/                       # 词条解析器核心
│   │   ├── src/
│   │   │   ├── extractors/
│   │   │   │   ├── base.extractor.ts
│   │   │   │   ├── vue.extractor.ts
│   │   │   │   └── react.extractor.ts
│   │   │   ├── ast/
│   │   │   │   ├── vue-ast-parser.ts
│   │   │   │   ├── jsx-ast-parser.ts
│   │   │   │   └── template-parser.ts
│   │   │   ├── utils/
│   │   │   │   ├── deduplicate.ts
│   │   │   │   └── normalize.ts
│   │   │   ├── types/
│   │   │   │   └── entry.ts
│   │   │   └── index.ts
│   │   ├── test/
│   │   │   └── fixtures/            # 测试用例
│   │   ├── package.json
│   │   └── tsconfig.json
│   │
│   ├── shared/                       # 共享代码
│   │   ├── src/
│   │   │   ├── types/
│   │   │   │   ├── project.ts
│   │   │   │   ├── entry.ts
│   │   │   │   ├── translation.ts
│   │   │   │   ├── version.ts
│   │   │   │   └── api.ts
│   │   │   ├── constants/
│   │   │   │   ├── locales.ts       # 支持的语言列表
│   │   │   │   ├── status.ts
│   │   │   │   └── frameworks.ts
│   │   │   ├── utils/
│   │   │   │   ├── date.ts
│   │   │   │   ├── string.ts
│   │   │   │   └── validation.ts
│   │   │   ├── validators/          # 通用验证器
│   │   │   └── index.ts
│   │   ├── package.json
│   │   └── tsconfig.json
│   │
│   └── sdk/                          # API SDK (供 CLI 使用)
│       ├── src/
│       │   ├── client.ts            # HTTP 客户端
│       │   ├── api/
│       │   │   ├── project.api.ts
│       │   │   ├── entry.api.ts
│       │   │   ├── translation.api.ts
│       │   │   └── version.api.ts
│       │   ├── types/
│       │   └── index.ts
│       ├── package.json
│       └── tsconfig.json
│
├── docs/                             # 文档
│   ├── getting-started.md
│   ├── cli-usage.md
│   ├── api-reference.md
│   └── deployment.md
│
├── .gitignore
├── .eslintrc.js
├── .prettierrc
├── package.json                      # 根 package.json
├── pnpm-workspace.yaml              # pnpm workspace 配置
├── turbo.json                        # Turborepo 配置
├── tsconfig.json                     # 根 TypeScript 配置
└── README.md
```

## package.json 示例

### 根 package.json

```json
{
  "name": "i18n-platform",
  "private": true,
  "workspaces": [
    "apps/*",
    "packages/*"
  ],
  "scripts": {
    "dev": "turbo run dev",
    "build": "turbo run build",
    "test": "turbo run test",
    "lint": "turbo run lint",
    "clean": "turbo run clean && rm -rf node_modules",
    "db:migrate": "cd apps/api && npm run migration:run",
    "db:seed": "cd apps/api && npm run seed"
  },
  "devDependencies": {
    "turbo": "^1.10.0",
    "@typescript-eslint/eslint-plugin": "^6.0.0",
    "@typescript-eslint/parser": "^6.0.0",
    "eslint": "^8.45.0",
    "prettier": "^3.0.0",
    "typescript": "^5.1.0"
  },
  "engines": {
    "node": ">=18.0.0",
    "pnpm": ">=8.0.0"
  },
  "packageManager": "pnpm@8.6.0"
}
```

### CLI package.json

```json
{
  "name": "@i18n-platform/cli",
  "version": "1.0.0",
  "description": "CLI tool for i18n platform",
  "bin": {
    "i18n-cli": "./bin/i18n-cli.js"
  },
  "main": "dist/index.js",
  "types": "dist/index.d.ts",
  "scripts": {
    "dev": "tsup --watch",
    "build": "tsup",
    "test": "vitest"
  },
  "dependencies": {
    "@i18n-platform/parser": "workspace:*",
    "@i18n-platform/sdk": "workspace:*",
    "@i18n-platform/shared": "workspace:*",
    "commander": "^11.0.0",
    "inquirer": "^9.2.0",
    "ora": "^7.0.0",
    "chalk": "^5.3.0",
    "cosmiconfig": "^8.3.0",
    "fast-glob": "^3.3.0",
    "fs-extra": "^11.1.1"
  },
  "devDependencies": {
    "tsup": "^7.2.0",
    "vitest": "^0.34.0"
  }
}
```

### Parser package.json

```json
{
  "name": "@i18n-platform/parser",
  "version": "1.0.0",
  "main": "dist/index.js",
  "types": "dist/index.d.ts",
  "scripts": {
    "dev": "tsup --watch",
    "build": "tsup",
    "test": "vitest"
  },
  "dependencies": {
    "@vue/compiler-sfc": "^3.3.0",
    "@babel/parser": "^7.22.0",
    "@babel/traverse": "^7.22.0",
    "@babel/types": "^7.22.0"
  }
}
```

## turbo.json 配置

```json
{
  "$schema": "https://turbo.build/schema.json",
  "globalDependencies": ["**/.env.*local"],
  "pipeline": {
    "build": {
      "dependsOn": ["^build"],
      "outputs": ["dist/**", ".next/**"]
    },
    "dev": {
      "cache": false,
      "persistent": true
    },
    "test": {
      "dependsOn": ["build"],
      "outputs": ["coverage/**"],
      "inputs": ["src/**/*.ts", "src/**/*.tsx", "test/**/*.ts"]
    },
    "lint": {
      "dependsOn": ["^build"]
    },
    "clean": {
      "cache": false
    }
  }
}
```

## pnpm-workspace.yaml

```yaml
packages:
  - 'apps/*'
  - 'packages/*'
```

## 依赖关系图

```
apps/web
  ├─→ @i18n-platform/shared
  └─→ @i18n-platform/sdk

apps/api
  ├─→ @i18n-platform/shared
  └─→ @i18n-platform/parser (用于验证提取结果)

packages/cli
  ├─→ @i18n-platform/parser
  ├─→ @i18n-platform/sdk
  └─→ @i18n-platform/shared

packages/sdk
  └─→ @i18n-platform/shared

packages/parser
  └─→ @i18n-platform/shared (types)

packages/shared
  (无依赖)
```

## 开发工作流

### 1. 安装依赖
```bash
pnpm install
```

### 2. 开发模式
```bash
# 启动所有服务
pnpm dev

# 或单独启动
pnpm --filter @i18n-platform/web dev
pnpm --filter @i18n-platform/api dev
```

### 3. 构建
```bash
# 构建所有包
pnpm build

# 构建特定包
pnpm --filter @i18n-platform/cli build
```

### 4. 测试
```bash
pnpm test
```

### 5. 发布 CLI
```bash
cd packages/cli
pnpm publish
```

## 环境变量管理

```
i18n-platform/
├── .env.example              # 示例环境变量
├── apps/
│   ├── web/
│   │   ├── .env.development
│   │   └── .env.production
│   └── api/
│       ├── .env.development
│       └── .env.production
```

### apps/api/.env.example
```env
# Database
DATABASE_URL=mysql://root:password@localhost:3306/i18n_platform
DATABASE_LOGGING=true

# Redis
REDIS_HOST=localhost
REDIS_PORT=6379

# JWT
JWT_SECRET=your-jwt-secret
JWT_EXPIRES_IN=7d

# AI Services
OPENAI_API_KEY=sk-xxx
GOOGLE_TRANSLATE_KEY=xxx
DEEPL_API_KEY=xxx

# Server
PORT=3000
NODE_ENV=development

# CORS
CORS_ORIGIN=http://localhost:5173
```

### apps/web/.env.example
```env
VITE_API_BASE_URL=http://localhost:3000/api
VITE_WS_URL=ws://localhost:3000
```
