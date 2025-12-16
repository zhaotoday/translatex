# Turborepo 项目结构

## 完整目录

```
translatex/
├── apps/
│   ├── web/                    # Vue 3 前端
│   │   ├── src/
│   │   │   ├── components/     # 公共组件
│   │   │   ├── views/          # 页面视图
│   │   │   ├── router/         # 路由配置
│   │   │   ├── store/          # Pinia Store
│   │   │   ├── api/            # API 调用
│   │   │   └── main.ts
│   │   ├── vite.config.ts
│   │   └── package.json
│   │
│   └── api/                    # NestJS 后端
│       ├── src/
│       │   ├── modules/
│       │   │   ├── auth/       # 认证模块
│       │   │   ├── project/    # 项目模块
│       │   │   ├── entry/      # 词条模块
│       │   │   ├── translation/# 翻译模块
│       │   │   ├── version/    # 版本模块
│       │   │   ├── ai/         # AI 翻译
│       │   │   └── queue/      # 队列处理
│       │   ├── app.module.ts
│       │   └── main.ts
│       └── package.json
│
├── packages/
│   ├── cli/                    # CLI 工具
│   │   ├── bin/i18n-cli.js
│   │   ├── src/
│   │   │   ├── commands/       # 命令实现
│   │   │   │   ├── init.ts
│   │   │   │   ├── extract.ts
│   │   │   │   ├── push.ts
│   │   │   │   ├── pull.ts
│   │   │   │   └── version/
│   │   │   └── config/
│   │   └── package.json
│   │
│   ├── parser/                 # AST 解析器
│   │   ├── src/
│   │   │   ├── extractors/
│   │   │   │   ├── vue.extractor.ts
│   │   │   │   └── react.extractor.ts
│   │   │   └── ast/
│   │   └── package.json
│   │
│   ├── sdk/                    # API SDK
│   │   ├── src/
│   │   │   ├── client.ts
│   │   │   └── api/
│   │   └── package.json
│   │
│   └── shared/                 # 共享代码
│       ├── src/
│       │   ├── types/          # 类型定义
│       │   ├── constants/      # 常量
│       │   └── utils/          # 工具函数
│       └── package.json
│
├── package.json                # 根 package.json
├── pnpm-workspace.yaml
├── turbo.json
└── tsconfig.json
```

## package.json 配置

### 根 package.json
```json
{
  "name": "translatex",
  "private": true,
  "workspaces": ["apps/*", "packages/*"],
  "scripts": {
    "dev": "turbo run dev",
    "build": "turbo run build",
    "test": "turbo run test"
  },
  "devDependencies": {
    "turbo": "^1.10.0",
    "typescript": "^5.1.0"
  },
  "packageManager": "pnpm@8.6.0"
}
```

### CLI package.json
```json
{
  "name": "@translatex/cli",
  "version": "1.0.0",
  "bin": {
    "i18n-cli": "./bin/i18n-cli.js"
  },
  "dependencies": {
    "@translatex/parser": "workspace:*",
    "@translatex/sdk": "workspace:*",
    "@translatex/shared": "workspace:*",
    "commander": "^11.0.0",
    "inquirer": "^9.2.0",
    "ora": "^7.0.0",
    "chalk": "^5.3.0"
  }
}
```

## turbo.json
```json
{
  "$schema": "https://turbo.build/schema.json",
  "pipeline": {
    "build": {
      "dependsOn": ["^build"],
      "outputs": ["dist/**"]
    },
    "dev": {
      "cache": false,
      "persistent": true
    },
    "test": {
      "dependsOn": ["build"]
    }
  }
}
```

## 依赖关系

```
apps/web
  ├─→ @translatex/shared
  └─→ @translatex/sdk

apps/api
  ├─→ @translatex/shared
  └─→ @translatex/parser

packages/cli
  ├─→ @translatex/parser
  ├─→ @translatex/sdk
  └─→ @translatex/shared

packages/sdk
  └─→ @translatex/shared

packages/parser
  └─→ @translatex/shared

packages/shared
  (无依赖)
```

## 开发工作流

```bash
# 安装依赖
pnpm install

# 开发模式
pnpm dev                              # 启动所有服务
pnpm --filter @translatex/web dev     # 仅启动前端
pnpm --filter @translatex/api dev     # 仅启动后端

# 构建
pnpm build                            # 构建所有包
pnpm --filter @translatex/cli build   # 构建 CLI

# 测试
pnpm test

# 发布
cd packages/cli && pnpm publish
```

## 环境变量

### apps/api/.env
```env
DATABASE_URL=mysql://root:password@localhost:3306/translatex
REDIS_HOST=localhost
REDIS_PORT=6379
JWT_SECRET=your-secret
OPENAI_API_KEY=sk-xxx
PORT=3000
```

### apps/web/.env
```env
VITE_API_BASE_URL=http://localhost:3000/api
VITE_WS_URL=ws://localhost:3000
```
