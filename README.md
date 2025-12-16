# 国际化管理平台设计文档

## 1. 项目概述

### 1.1 项目背景
构建一个完整的国际化(i18n)管理平台，支持从源代码提取词条、AI 翻译、版本管理、多项目管理等功能。

### 1.2 核心功能
- **词条提取**：从 Vue/React 项目中自动提取需要翻译的文案
- **词条管理**：平台化管理所有翻译词条
- **AI 翻译**：集成 AI 能力自动翻译词条
- **版本控制**：支持词条的版本管理，与项目分支对应
- **CLI 工具**：提供命令行工具，方便开发者使用

### 1.3 技术栈
- **后端**: NestJS + Sequelize + MySQL/PostgreSQL
- **前端**: Vue 3 + TypeScript + Element Plus/Ant Design Vue
- **CLI**: Node.js + Commander.js
- **Monorepo**: Turborepo
- **AI 翻译**: OpenAI API / 其他翻译服务
- **其他**: Docker、Redis、OSS(对象存储)

---

## 2. 系统架构

### 2.1 整体架构图

```
┌─────────────────────────────────────────────────────────────┐
│                        开发者本地                              │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  前端项目 (Vue/React)                                   │   │
│  │  ├── src/                                             │   │
│  │  ├── locales/  (翻译文件目录)                           │   │
│  │  └── i18n.config.js                                   │   │
│  └──────────────────────────────────────────────────────┘   │
│                          ↕                                   │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  @i18n-platform/cli  (命令行工具)                       │   │
│  │  ├── extract  (提取词条)                                │   │
│  │  ├── push     (上传词条)                                │   │
│  │  └── pull     (下载译文)                                │   │
│  └──────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
                          ↕ HTTPS
┌─────────────────────────────────────────────────────────────┐
│                      i18n 管理平台                            │
│  ┌──────────────────┐    ┌──────────────────┐               │
│  │  前端应用 (Vue)    │←→│  后端 API (Nest)   │               │
│  │  - 项目管理        │    │  - RESTful API    │               │
│  │  - 词条管理        │    │  - WebSocket      │               │
│  │  - 翻译界面        │    │  - 任务队列        │               │
│  │  - 版本控制        │    └──────────────────┘               │
│  └──────────────────┘              ↕                         │
│                        ┌──────────────────────┐              │
│                        │  MySQL/PostgreSQL    │              │
│                        │  - 项目表             │              │
│                        │  - 词条表             │              │
│                        │  - 翻译表             │              │
│                        │  - 版本表             │              │
│                        └──────────────────────┘              │
│         ↕                         ↕                          │
│  ┌──────────────┐      ┌──────────────────────┐             │
│  │  Redis       │      │  AI 翻译服务          │             │
│  │  - 缓存       │      │  - OpenAI            │             │
│  │  - 队列       │      │  - Google Translate  │             │
│  └──────────────┘      │  - DeepL             │             │
│                        └──────────────────────┘              │
└─────────────────────────────────────────────────────────────┘
```

### 2.2 Turborepo 项目结构

```
i18n-platform/
├── package.json
├── turbo.json
├── pnpm-workspace.yaml
├── apps/
│   ├── web/                      # 前端管理平台 (Vue 3)
│   │   ├── src/
│   │   ├── package.json
│   │   └── vite.config.ts
│   │
│   └── api/                      # 后端 API (NestJS)
│       ├── src/
│       │   ├── modules/
│       │   │   ├── project/      # 项目模块
│       │   │   ├── entry/        # 词条模块
│       │   │   ├── translation/  # 翻译模块
│       │   │   ├── version/      # 版本模块
│       │   │   └── ai/           # AI 翻译模块
│       │   ├── main.ts
│       │   └── app.module.ts
│       └── package.json
│
├── packages/
│   ├── cli/                      # CLI 工具
│   │   ├── src/
│   │   │   ├── commands/
│   │   │   │   ├── extract.ts    # 提取命令
│   │   │   │   ├── push.ts       # 上传命令
│   │   │   │   └── pull.ts       # 下载命令
│   │   │   ├── parsers/
│   │   │   │   ├── vue.ts        # Vue 解析器
│   │   │   │   └── react.ts      # React 解析器
│   │   │   └── index.ts
│   │   └── package.json
│   │
│   ├── shared/                   # 共享代码
│   │   ├── src/
│   │   │   ├── types/            # TypeScript 类型
│   │   │   ├── constants/        # 常量
│   │   │   └── utils/            # 工具函数
│   │   └── package.json
│   │
│   ├── parser/                   # 词条解析器核心
│   │   ├── src/
│   │   │   ├── extractors/       # 提取器
│   │   │   ├── ast/              # AST 解析
│   │   │   └── index.ts
│   │   └── package.json
│   │
│   └── sdk/                      # API SDK
│       ├── src/
│       │   ├── client.ts
│       │   └── api/
│       └── package.json
│
├── docs/                         # 文档
└── README.md
```

---

## 3. 数据库设计

### 3.1 ER 图概述

```
┌─────────────┐       ┌─────────────┐       ┌─────────────┐
│   Project   │──1:N──│    Entry    │──1:N──│ Translation │
│   项目表     │       │   词条表     │       │   翻译表     │
└─────────────┘       └─────────────┘       └─────────────┘
                              │
                            1:N
                              │
                      ┌─────────────┐
                      │   Version   │
                      │   版本表     │
                      └─────────────┘
```

### 3.2 详细表结构

#### 3.2.1 用户表 (users)
```sql
CREATE TABLE users (
  id BIGINT PRIMARY KEY AUTO_INCREMENT,
  username VARCHAR(50) UNIQUE NOT NULL,
  email VARCHAR(100) UNIQUE NOT NULL,
  password VARCHAR(255) NOT NULL,
  role ENUM('admin', 'developer', 'translator') DEFAULT 'developer',
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);
```

#### 3.2.2 项目表 (projects)
```sql
CREATE TABLE projects (
  id BIGINT PRIMARY KEY AUTO_INCREMENT,
  name VARCHAR(100) NOT NULL,
  slug VARCHAR(100) UNIQUE NOT NULL COMMENT '项目唯一标识',
  description TEXT,
  repo_url VARCHAR(255) COMMENT 'Git 仓库地址',
  framework ENUM('vue', 'react', 'angular') NOT NULL,
  default_locale VARCHAR(10) DEFAULT 'zh-CN' COMMENT '默认语言',
  supported_locales JSON COMMENT '支持的语言列表 ["en-US", "zh-CN"]',
  api_key VARCHAR(64) UNIQUE NOT NULL COMMENT 'API 密钥',
  owner_id BIGINT NOT NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  FOREIGN KEY (owner_id) REFERENCES users(id)
);
```

#### 3.2.3 词条表 (entries)
```sql
CREATE TABLE entries (
  id BIGINT PRIMARY KEY AUTO_INCREMENT,
  project_id BIGINT NOT NULL,
  entry_key VARCHAR(255) NOT NULL COMMENT '词条 key，如 home.welcome',
  source_text TEXT NOT NULL COMMENT '源文本',
  context VARCHAR(500) COMMENT '上下文，帮助翻译理解',
  file_path VARCHAR(500) COMMENT '源文件路径',
  line_number INT COMMENT '行号',
  namespace VARCHAR(100) DEFAULT 'common' COMMENT '命名空间',
  status ENUM('pending', 'translating', 'translated', 'approved') DEFAULT 'pending',
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  UNIQUE KEY uk_project_key (project_id, entry_key),
  FOREIGN KEY (project_id) REFERENCES projects(id) ON DELETE CASCADE,
  INDEX idx_project_status (project_id, status)
);
```

#### 3.2.4 翻译表 (translations)
```sql
CREATE TABLE translations (
  id BIGINT PRIMARY KEY AUTO_INCREMENT,
  entry_id BIGINT NOT NULL,
  locale VARCHAR(10) NOT NULL COMMENT '语言代码',
  translated_text TEXT NOT NULL COMMENT '翻译文本',
  translation_type ENUM('manual', 'ai', 'import') DEFAULT 'manual',
  translator_id BIGINT COMMENT '翻译者 ID',
  ai_provider VARCHAR(50) COMMENT 'AI 提供商，如 openai',
  quality_score DECIMAL(3,2) COMMENT '翻译质量评分 0-1',
  is_reviewed BOOLEAN DEFAULT FALSE COMMENT '是否已审核',
  reviewer_id BIGINT COMMENT '审核者 ID',
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  FOREIGN KEY (entry_id) REFERENCES entries(id) ON DELETE CASCADE,
  FOREIGN KEY (translator_id) REFERENCES users(id),
  FOREIGN KEY (reviewer_id) REFERENCES users(id),
  UNIQUE KEY uk_entry_locale (entry_id, locale),
  INDEX idx_locale (locale)
);
```

#### 3.2.5 版本表 (versions)
```sql
CREATE TABLE versions (
  id BIGINT PRIMARY KEY AUTO_INCREMENT,
  project_id BIGINT NOT NULL,
  version_name VARCHAR(100) NOT NULL COMMENT '版本名称，如 v1.0.0',
  branch_name VARCHAR(100) NOT NULL COMMENT '对应的 Git 分支',
  commit_hash VARCHAR(40) COMMENT 'Git commit hash',
  is_master BOOLEAN DEFAULT FALSE COMMENT '是否为主版本（最新可用版本）',
  snapshot JSON COMMENT '该版本的词条快照',
  description TEXT,
  created_by BIGINT,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  FOREIGN KEY (project_id) REFERENCES projects(id) ON DELETE CASCADE,
  FOREIGN KEY (created_by) REFERENCES users(id),
  UNIQUE KEY uk_project_version (project_id, version_name),
  INDEX idx_project_master (project_id, is_master)
);
```

#### 3.2.6 词条版本关联表 (entry_versions)
```sql
CREATE TABLE entry_versions (
  id BIGINT PRIMARY KEY AUTO_INCREMENT,
  entry_id BIGINT NOT NULL,
  version_id BIGINT NOT NULL,
  source_text TEXT NOT NULL COMMENT '该版本的源文本',
  translations JSON COMMENT '该版本的所有翻译 {"en-US": "...", "zh-CN": "..."}',
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (entry_id) REFERENCES entries(id) ON DELETE CASCADE,
  FOREIGN KEY (version_id) REFERENCES versions(id) ON DELETE CASCADE,
  UNIQUE KEY uk_entry_version (entry_id, version_id)
);
```

#### 3.2.7 翻译任务表 (translation_jobs)
```sql
CREATE TABLE translation_jobs (
  id BIGINT PRIMARY KEY AUTO_INCREMENT,
  project_id BIGINT NOT NULL,
  source_locale VARCHAR(10) NOT NULL,
  target_locale VARCHAR(10) NOT NULL,
  total_entries INT DEFAULT 0,
  translated_entries INT DEFAULT 0,
  status ENUM('pending', 'processing', 'completed', 'failed') DEFAULT 'pending',
  error_message TEXT,
  started_at TIMESTAMP NULL,
  completed_at TIMESTAMP NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (project_id) REFERENCES projects(id) ON DELETE CASCADE,
  INDEX idx_status (status)
);
```

---

## 4. CLI 工具设计

### 4.1 命令结构

```bash
# 初始化配置
i18n-cli init

# 提取词条
i18n-cli extract [options]

# 上传词条到平台
i18n-cli push [options]

# 下载翻译后的语言包
i18n-cli pull [options]

# 查看同步状态
i18n-cli status

# 创建新版本
i18n-cli version:create <version-name>

# 设置主版本
i18n-cli version:master <version-name>
```

### 4.2 配置文件 (i18n.config.js)

```javascript
module.exports = {
  // 项目唯一标识
  projectId: 'your-project-slug',
  
  // API 密钥
  apiKey: 'your-api-key',
  
  // 平台地址
  apiEndpoint: 'https://i18n-platform.com/api',
  
  // 项目框架
  framework: 'vue', // 'vue' | 'react'
  
  // 源语言
  defaultLocale: 'zh-CN',
  
  // 目标语言
  targetLocales: ['en-US', 'ja-JP', 'ko-KR'],
  
  // 词条提取配置
  extract: {
    // 扫描的文件目录
    include: ['src/**/*.{vue,js,ts,jsx,tsx}'],
    
    // 排除的目录
    exclude: ['node_modules/**', 'dist/**'],
    
    // 提取函数名称
    functionNames: ['$t', 't', 'i18n.t'],
    
    // 提取的组件属性
    componentProps: ['i18n-t', 'v-t'],
    
    // 输出目录
    outputDir: './locales/extracted',
  },
  
  // 语言包输出配置
  output: {
    // 语言包目录
    localesDir: './src/locales',
    
    // 文件格式
    format: 'json', // 'json' | 'yaml' | 'js'
    
    // 是否扁平化
    flat: false,
    
    // 文件命名规则
    filename: '[locale].json', // [locale] 会被替换为 zh-CN, en-US 等
  },
  
  // 版本控制
  version: {
    // 当前工作分支
    branch: 'develop',
    
    // 是否自动创建版本
    autoVersion: false,
  },
};
```

### 4.3 提取器实现思路

#### 4.3.1 Vue 提取器
使用 `@vue/compiler-sfc` 解析 Vue 文件：

```typescript
// packages/parser/src/extractors/vue.ts
import { parse } from '@vue/compiler-sfc';
import * as babelParser from '@babel/parser';
import traverse from '@babel/traverse';

export class VueExtractor {
  extract(fileContent: string, filePath: string) {
    const entries = [];
    
    // 解析 Vue SFC
    const { descriptor } = parse(fileContent);
    
    // 1. 提取 <template> 中的词条
    if (descriptor.template) {
      // 使用正则或 AST 查找 {{ $t('xxx') }} 或 v-t="'xxx'"
      entries.push(...this.extractFromTemplate(descriptor.template.content));
    }
    
    // 2. 提取 <script> 中的词条
    if (descriptor.script || descriptor.scriptSetup) {
      const scriptContent = descriptor.script?.content || descriptor.scriptSetup.content;
      entries.push(...this.extractFromScript(scriptContent));
    }
    
    return entries;
  }
  
  private extractFromScript(code: string) {
    const entries = [];
    const ast = babelParser.parse(code, {
      sourceType: 'module',
      plugins: ['typescript', 'jsx'],
    });
    
    // 遍历 AST，查找 $t('xxx') 或 t('xxx') 调用
    traverse(ast, {
      CallExpression(path) {
        const { callee, arguments: args } = path.node;
        
        if (
          (callee.type === 'Identifier' && ['t', '$t'].includes(callee.name)) ||
          (callee.type === 'MemberExpression' && 
           callee.property.name === 't')
        ) {
          if (args[0]?.type === 'StringLiteral') {
            entries.push({
              key: args[0].value,
              context: args[1]?.value, // 第二个参数可能是上下文
            });
          }
        }
      },
    });
    
    return entries;
  }
}
```

#### 4.3.2 React 提取器
类似方式，使用 `@babel/parser` 解析 JSX：

```typescript
// packages/parser/src/extractors/react.ts
export class ReactExtractor {
  extract(fileContent: string, filePath: string) {
    const ast = babelParser.parse(fileContent, {
      sourceType: 'module',
      plugins: ['typescript', 'jsx'],
    });
    
    const entries = [];
    
    traverse(ast, {
      // 提取 t('xxx') 调用
      CallExpression(path) {
        // ... 类似 Vue 的逻辑
      },
      
      // 提取 <Trans i18nKey="xxx" /> 组件
      JSXElement(path) {
        const openingElement = path.node.openingElement;
        if (openingElement.name.name === 'Trans') {
          const i18nKeyAttr = openingElement.attributes.find(
            attr => attr.name?.name === 'i18nKey'
          );
          if (i18nKeyAttr?.value?.value) {
            entries.push({ key: i18nKeyAttr.value.value });
          }
        }
      },
    });
    
    return entries;
  }
}
```

### 4.4 CLI 命令实现示例

```typescript
// packages/cli/src/commands/extract.ts
import { Command } from 'commander';
import { loadConfig } from '../config';
import { VueExtractor } from '@i18n-platform/parser';
import glob from 'fast-glob';

export const extractCommand = new Command('extract')
  .description('Extract i18n entries from source code')
  .option('-c, --config <path>', 'Config file path', './i18n.config.js')
  .option('-o, --output <path>', 'Output directory')
  .action(async (options) => {
    const config = await loadConfig(options.config);
    const extractor = new VueExtractor(); // 根据 framework 选择
    
    // 扫描文件
    const files = await glob(config.extract.include, {
      ignore: config.extract.exclude,
    });
    
    const allEntries = [];
    
    for (const file of files) {
      const content = await fs.readFile(file, 'utf-8');
      const entries = extractor.extract(content, file);
      allEntries.push(...entries);
    }
    
    // 去重、整理
    const uniqueEntries = deduplicateEntries(allEntries);
    
    // 输出到文件
    const outputPath = options.output || config.extract.outputDir;
    await fs.writeJSON(`${outputPath}/extracted.json`, uniqueEntries, {
      spaces: 2,
    });
    
    console.log(`✅ Extracted ${uniqueEntries.length} entries`);
  });
```

```typescript
// packages/cli/src/commands/push.ts
import { Command } from 'commander';
import { I18nApiClient } from '@i18n-platform/sdk';

export const pushCommand = new Command('push')
  .description('Push extracted entries to platform')
  .option('-f, --file <path>', 'Entries file path', './locales/extracted/extracted.json')
  .action(async (options) => {
    const config = await loadConfig();
    const client = new I18nApiClient({
      apiKey: config.apiKey,
      endpoint: config.apiEndpoint,
    });
    
    // 读取提取的词条
    const entries = await fs.readJSON(options.file);
    
    // 上传到平台
    const spinner = ora('Uploading entries...').start();
    
    try {
      const result = await client.entries.batchCreate(
        config.projectId,
        entries,
        {
          branch: config.version.branch,
        }
      );
      
      spinner.succeed(`✅ Uploaded ${result.created} new entries, ${result.updated} updated`);
    } catch (error) {
      spinner.fail('❌ Upload failed');
      console.error(error.message);
    }
  });
```

```typescript
// packages/cli/src/commands/pull.ts
import { Command } from 'commander';

export const pullCommand = new Command('pull')
  .description('Pull translations from platform')
  .option('-l, --locales <locales>', 'Locales to pull (comma-separated)')
  .option('--version <version>', 'Version to pull (default: master)')
  .action(async (options) => {
    const config = await loadConfig();
    const client = new I18nApiClient(config);
    
    const locales = options.locales?.split(',') || config.targetLocales;
    const version = options.version || 'master';
    
    for (const locale of locales) {
      const spinner = ora(`Pulling ${locale}...`).start();
      
      try {
        // 从平台拉取翻译
        const translations = await client.translations.pull(
          config.projectId,
          locale,
          { version }
        );
        
        // 写入本地文件
        const outputPath = config.output.localesDir;
        const filename = config.output.filename.replace('[locale]', locale);
        const filePath = path.join(outputPath, filename);
        
        if (config.output.format === 'json') {
          await fs.writeJSON(filePath, translations, { spaces: 2 });
        } else if (config.output.format === 'yaml') {
          await fs.writeFile(filePath, yaml.dump(translations));
        }
        
        spinner.succeed(`✅ ${locale} pulled successfully`);
      } catch (error) {
        spinner.fail(`❌ Failed to pull ${locale}`);
        console.error(error.message);
      }
    }
  });
```

---

## 5. 后端 API 设计

### 5.1 RESTful API 接口列表

#### 5.1.1 项目管理

```typescript
// 创建项目
POST /api/projects
Body: {
  name: string;
  slug: string;
  framework: 'vue' | 'react';
  defaultLocale: string;
  supportedLocales: string[];
}

// 获取项目列表
GET /api/projects
Query: { page, limit, keyword }

// 获取项目详情
GET /api/projects/:projectId

// 更新项目
PATCH /api/projects/:projectId

// 删除项目
DELETE /api/projects/:projectId

// 生成 API Key
POST /api/projects/:projectId/api-key/regenerate
```

#### 5.1.2 词条管理

```typescript
// 批量创建/更新词条
POST /api/projects/:projectId/entries/batch
Body: {
  entries: Array<{
    key: string;
    sourceText: string;
    context?: string;
    filePath?: string;
    lineNumber?: number;
  }>;
  branch?: string;
}

// 获取词条列表
GET /api/projects/:projectId/entries
Query: { 
  page, 
  limit, 
  keyword, 
  status, 
  namespace,
  locale // 可选，如果提供则返回对应语言的翻译
}

// 获取单个词条
GET /api/projects/:projectId/entries/:entryId

// 更新词条
PATCH /api/projects/:projectId/entries/:entryId

// 删除词条
DELETE /api/projects/:projectId/entries/:entryId
```

#### 5.1.3 翻译管理

```typescript
// 创建/更新翻译
PUT /api/entries/:entryId/translations/:locale
Body: {
  translatedText: string;
  translationType: 'manual' | 'ai';
}

// 批量 AI 翻译
POST /api/projects/:projectId/translations/ai-translate
Body: {
  sourceLocale: string;
  targetLocales: string[];
  entryIds?: number[]; // 可选，不提供则翻译所有未翻译的
  aiProvider?: 'openai' | 'google' | 'deepl';
}

// 获取翻译进度
GET /api/projects/:projectId/translations/progress
Query: { locale }

// 审核翻译
POST /api/translations/:translationId/review
Body: {
  approved: boolean;
  comment?: string;
}
```

#### 5.1.4 版本管理

```typescript
// 创建版本
POST /api/projects/:projectId/versions
Body: {
  versionName: string;
  branchName: string;
  commitHash?: string;
  description?: string;
}

// 获取版本列表
GET /api/projects/:projectId/versions

// 设置主版本
POST /api/projects/:projectId/versions/:versionId/set-master

// 获取版本详情（包含词条快照）
GET /api/projects/:projectId/versions/:versionId

// 比较两个版本
GET /api/projects/:projectId/versions/compare
Query: { fromVersion, toVersion }
```

#### 5.1.5 导出/导入

```typescript
// 导出语言包（供 CLI pull 使用）
GET /api/projects/:projectId/export/:locale
Query: { version?, format?: 'json' | 'yaml' }

// 导出所有语言包（ZIP）
GET /api/projects/:projectId/export/all
Query: { version?, format? }

// 导入语言包
POST /api/projects/:projectId/import
Body: FormData (file upload)
```

### 5.2 NestJS 模块结构

```typescript
// apps/api/src/modules/entry/entry.controller.ts
@Controller('projects/:projectId/entries')
@UseGuards(ApiKeyGuard)
export class EntryController {
  constructor(private readonly entryService: EntryService) {}
  
  @Post('batch')
  async batchCreateOrUpdate(
    @Param('projectId') projectId: number,
    @Body() dto: BatchCreateEntriesDto,
  ) {
    return this.entryService.batchCreateOrUpdate(projectId, dto);
  }
  
  @Get()
  async findAll(
    @Param('projectId') projectId: number,
    @Query() query: FindEntriesDto,
  ) {
    return this.entryService.findAll(projectId, query);
  }
}
```

```typescript
// apps/api/src/modules/entry/entry.service.ts
@Injectable()
export class EntryService {
  constructor(
    @InjectModel(Entry) private entryModel: typeof Entry,
    @InjectModel(Translation) private translationModel: typeof Translation,
  ) {}
  
  async batchCreateOrUpdate(
    projectId: number,
    dto: BatchCreateEntriesDto,
  ) {
    const results = { created: 0, updated: 0, errors: [] };
    
    for (const entryData of dto.entries) {
      try {
        const [entry, created] = await this.entryModel.findOrCreate({
          where: {
            projectId,
            entryKey: entryData.key,
          },
          defaults: {
            sourceText: entryData.sourceText,
            context: entryData.context,
            filePath: entryData.filePath,
            lineNumber: entryData.lineNumber,
          },
        });
        
        if (!created && entry.sourceText !== entryData.sourceText) {
          // 源文本有变化，更新
          await entry.update({ sourceText: entryData.sourceText });
          results.updated++;
        } else if (created) {
          results.created++;
        }
      } catch (error) {
        results.errors.push({
          key: entryData.key,
          error: error.message,
        });
      }
    }
    
    return results;
  }
}
```

### 5.3 AI 翻译服务

```typescript
// apps/api/src/modules/ai/ai.service.ts
@Injectable()
export class AiTranslationService {
  constructor(
    private readonly openaiService: OpenAiService,
    private readonly queueService: QueueService,
  ) {}
  
  async translateBatch(
    projectId: number,
    sourceLocale: string,
    targetLocales: string[],
    entryIds?: number[],
  ) {
    // 创建翻译任务
    const jobs = await Promise.all(
      targetLocales.map(locale =>
        this.createTranslationJob(projectId, sourceLocale, locale, entryIds)
      )
    );
    
    // 加入队列异步处理
    for (const job of jobs) {
      await this.queueService.addJob('translation', {
        jobId: job.id,
        projectId,
        sourceLocale,
        targetLocale: job.targetLocale,
        entryIds,
      });
    }
    
    return jobs;
  }
  
  async processTranslationJob(jobData: any) {
    const { jobId, projectId, sourceLocale, targetLocale, entryIds } = jobData;
    
    // 更新任务状态
    await this.updateJobStatus(jobId, 'processing');
    
    try {
      // 获取待翻译的词条
      const entries = await this.getEntriesToTranslate(
        projectId,
        targetLocale,
        entryIds
      );
      
      let translated = 0;
      
      // 批量翻译（每次翻译 20 条）
      for (let i = 0; i < entries.length; i += 20) {
        const batch = entries.slice(i, i + 20);
        
        const translations = await this.openaiService.translateBatch(
          batch.map(e => e.sourceText),
          sourceLocale,
          targetLocale,
        );
        
        // 保存翻译结果
        await Promise.all(
          batch.map((entry, index) =>
            this.saveTranslation(entry.id, targetLocale, translations[index])
          )
        );
        
        translated += batch.length;
        
        // 更新进度
        await this.updateJobProgress(jobId, translated, entries.length);
      }
      
      await this.updateJobStatus(jobId, 'completed');
    } catch (error) {
      await this.updateJobStatus(jobId, 'failed', error.message);
    }
  }
}
```

```typescript
// apps/api/src/modules/ai/providers/openai.service.ts
@Injectable()
export class OpenAiService {
  private openai: OpenAI;
  
  constructor(private configService: ConfigService) {
    this.openai = new OpenAI({
      apiKey: configService.get('OPENAI_API_KEY'),
    });
  }
  
  async translateBatch(
    texts: string[],
    sourceLang: string,
    targetLang: string,
  ): Promise<string[]> {
    const prompt = `
You are a professional translator. Translate the following texts from ${sourceLang} to ${targetLang}.
Keep the same format and placeholders (like {variable}, {{interpolation}}).
Return only the translations as a JSON array.

Texts to translate:
${JSON.stringify(texts, null, 2)}
    `;
    
    const response = await this.openai.chat.completions.create({
      model: 'gpt-4',
      messages: [{ role: 'user', content: prompt }],
      temperature: 0.3,
    });
    
    const content = response.choices[0].message.content;
    return JSON.parse(content);
  }
}
```

---

## 6. 前端界面设计

### 6.1 页面结构

```
apps/web/src/
├── views/
│   ├── Dashboard.vue           # 仪表盘
│   ├── Projects/
│   │   ├── ProjectList.vue     # 项目列表
│   │   ├── ProjectDetail.vue   # 项目详情
│   │   └── ProjectSettings.vue # 项目设置
│   ├── Entries/
│   │   ├── EntryList.vue       # 词条列表
│   │   └── EntryEditor.vue     # 词条编辑器
│   ├── Translations/
│   │   ├── TranslationBoard.vue # 翻译工作台
│   │   └── BatchTranslate.vue   # 批量翻译
│   └── Versions/
│       ├── VersionList.vue      # 版本列表
│       └── VersionCompare.vue   # 版本对比
```

### 6.2 核心页面功能

#### 6.2.1 词条列表 (EntryList.vue)

功能：
- 表格展示所有词条（key、源文本、翻译状态）
- 支持搜索、筛选（按状态、命名空间）
- 支持批量操作（批量翻译、导出）
- 点击词条进入编辑器

```vue
<template>
  <div class="entry-list">
    <el-card>
      <div class="toolbar">
        <el-input 
          v-model="searchKeyword" 
          placeholder="搜索词条..."
          @input="handleSearch"
        />
        
        <el-select v-model="statusFilter">
          <el-option label="全部" value="" />
          <el-option label="待翻译" value="pending" />
          <el-option label="已翻译" value="translated" />
        </el-select>
        
        <el-button type="primary" @click="handleBatchTranslate">
          批量 AI 翻译
        </el-button>
      </div>
      
      <el-table :data="entries" @selection-change="handleSelectionChange">
        <el-table-column type="selection" />
        <el-table-column prop="entryKey" label="Key" />
        <el-table-column prop="sourceText" label="源文本" />
        
        <el-table-column label="翻译进度">
          <template #default="{ row }">
            {{ row.translatedCount }} / {{ targetLocales.length }}
          </template>
        </el-table-column>
        
        <el-table-column label="状态">
          <template #default="{ row }">
            <el-tag :type="getStatusType(row.status)">
              {{ row.status }}
            </el-tag>
          </template>
        </el-table-column>
        
        <el-table-column label="操作">
          <template #default="{ row }">
            <el-button link @click="handleEdit(row)">编辑</el-button>
          </template>
        </el-table-column>
      </el-table>
    </el-card>
  </div>
</template>
```

#### 6.2.2 翻译工作台 (TranslationBoard.vue)

功能：
- 左侧词条列表，右侧翻译编辑器
- 支持多语言并排显示
- 支持 AI 翻译建议
- 翻译历史记录

```vue
<template>
  <div class="translation-board">
    <div class="entry-sidebar">
      <el-menu>
        <el-menu-item 
          v-for="entry in entries" 
          :key="entry.id"
          @click="selectEntry(entry)"
        >
          <div class="entry-item">
            <div class="entry-key">{{ entry.key }}</div>
            <div class="entry-text">{{ entry.sourceText }}</div>
          </div>
        </el-menu-item>
      </el-menu>
    </div>
    
    <div class="translation-editor">
      <div class="source-section">
        <h3>源文本 ({{ sourceLocale }})</h3>
        <div class="source-text">{{ currentEntry?.sourceText }}</div>
      </div>
      
      <el-divider />
      
      <div v-for="locale in targetLocales" :key="locale" class="translation-section">
        <div class="translation-header">
          <h4>{{ locale }}</h4>
          <el-button 
            size="small" 
            @click="aiTranslate(locale)"
            :loading="aiLoading[locale]"
          >
            AI 翻译
          </el-button>
        </div>
        
        <el-input
          v-model="translations[locale]"
          type="textarea"
          :rows="3"
          @change="handleTranslationChange(locale)"
        />
        
        <!-- AI 建议 -->
        <div v-if="aiSuggestions[locale]" class="ai-suggestion">
          <span>AI 建议：</span>
          <span class="suggestion-text">{{ aiSuggestions[locale] }}</span>
          <el-button link @click="useSuggestion(locale)">使用</el-button>
        </div>
      </div>
    </div>
  </div>
</template>
```

#### 6.2.3 版本管理 (VersionList.vue)

功能：
- 显示所有版本
- 创建新版本（基于当前词条状态）
- 标记主版本
- 版本对比

```vue
<template>
  <div class="version-list">
    <el-button type="primary" @click="createVersionDialogVisible = true">
      创建新版本
    </el-button>
    
    <el-table :data="versions">
      <el-table-column prop="versionName" label="版本名称" />
      <el-table-column prop="branchName" label="分支" />
      <el-table-column label="状态">
        <template #default="{ row }">
          <el-tag v-if="row.isMaster" type="success">主版本</el-tag>
        </template>
      </el-table-column>
      <el-table-column prop="createdAt" label="创建时间" />
      <el-table-column label="操作">
        <template #default="{ row }">
          <el-button 
            v-if="!row.isMaster" 
            link 
            @click="setMaster(row)"
          >
            设为主版本
          </el-button>
          <el-button link @click="viewVersion(row)">查看</el-button>
        </template>
      </el-table-column>
    </el-table>
  </div>
</template>
```

---

## 7. 工作流程

### 7.1 完整流程图

```
┌─────────────────────────────────────────────────────────────┐
│                   开发者工作流程                               │
└─────────────────────────────────────────────────────────────┘

1. 初始化项目
   ┌────────────────┐
   │ i18n-cli init  │  创建配置文件，绑定项目
   └────────────────┘
           │
           ↓

2. 开发阶段 - 提取词条
   ┌─────────────────┐
   │ 编写代码，使用     │  代码中使用 $t('home.welcome')
   │ $t() 函数        │
   └─────────────────┘
           │
           ↓
   ┌─────────────────┐
   │ i18n-cli extract│  扫描源代码，提取所有词条
   └─────────────────┘  输出: locales/extracted/extracted.json
           │
           ↓

3. 上传到平台
   ┌─────────────────┐
   │ i18n-cli push   │  上传词条到平台
   └─────────────────┘  自动关联到当前分支
           │
           ↓

4. 平台翻译（管理员/翻译人员）
   ┌─────────────────────────────────────┐
   │  在 Web 平台上                        │
   │  - 查看新增的词条                     │
   │  - 使用 AI 批量翻译                   │
   │  - 人工审核、修改翻译                 │
   │  - 标记翻译完成                       │
   └─────────────────────────────────────┘
           │
           ↓

5. 下载翻译
   ┌─────────────────┐
   │ i18n-cli pull   │  拉取所有语言的翻译
   └─────────────────┘  输出: src/locales/{locale}.json
           │
           ↓

6. 版本发布
   ┌──────────────────────────┐
   │ i18n-cli version:create  │  创建版本快照
   │   v1.0.0                 │
   └──────────────────────────┘
           │
           ↓
   ┌──────────────────────────┐
   │ i18n-cli version:master  │  标记为主版本
   │   v1.0.0                 │
   └──────────────────────────┘
           │
           ↓

7. 应用运行
   ┌─────────────────┐
   │ 前端应用读取     │  根据用户语言加载对应的翻译文件
   │ locales/*.json  │
   └─────────────────┘
```

### 7.2 AI 翻译流程

```
┌─────────────────────────────────────────────────────────────┐
│                    AI 翻译流程                                │
└─────────────────────────────────────────────────────────────┘

1. 触发翻译
   ┌─────────────────────────┐
   │ 用户在 Web 平台点击       │
   │ "批量 AI 翻译" 按钮       │
   └─────────────────────────┘
           │
           ↓

2. 创建翻译任务
   ┌─────────────────────────────────┐
   │ 后端创建 translation_jobs 记录   │
   │ status: pending                 │
   └─────────────────────────────────┘
           │
           ↓

3. 加入队列
   ┌─────────────────────────────────┐
   │ 任务加入 Redis 队列              │
   │ (使用 Bull 或 BullMQ)            │
   └─────────────────────────────────┘
           │
           ↓

4. Worker 处理
   ┌─────────────────────────────────┐
   │ Worker 从队列中取出任务          │
   │ 更新 status: processing         │
   └─────────────────────────────────┘
           │
           ↓

5. 调用 AI API
   ┌─────────────────────────────────┐
   │ 分批调用 OpenAI/Google API      │
   │ 每批 20 条词条                   │
   └─────────────────────────────────┘
           │
           ↓

6. 保存结果
   ┌─────────────────────────────────┐
   │ 将翻译结果写入 translations 表   │
   │ translation_type: 'ai'          │
   └─────────────────────────────────┘
           │
           ↓

7. 更新进度
   ┌─────────────────────────────────┐
   │ 更新 job 的进度                  │
   │ translated_entries / total      │
   │ 通过 WebSocket 推送给前端        │
   └─────────────────────────────────┘
           │
           ↓

8. 完成
   ┌─────────────────────────────────┐
   │ status: completed               │
   │ 用户可审核 AI 翻译结果           │
   └─────────────────────────────────┘
```

### 7.3 版本控制流程

```
开发分支: feature-login
   │
   ├─ 提取词条 (10 个新词条)
   ├─ push 到平台
   ├─ 翻译完成
   └─ pull 下来
   
合并到 develop 分支
   │
   └─ 创建版本: v1.1.0-beta (branch: develop)
   
测试通过，合并到 master
   │
   └─ 创建版本: v1.1.0 (branch: master)
       │
       └─ 标记为主版本 (isMaster: true)
   
之后所有 CLI pull 默认拉取 v1.1.0 的翻译
```

---

## 8. 技术实现要点

### 8.1 推荐的开源库

#### CLI 工具
- **commander**: 命令行框架
- **inquirer**: 交互式命令行提示
- **ora**: 加载动画
- **chalk**: 终端颜色输出
- **cosmiconfig**: 配置文件加载

#### 词条提取
- **@vue/compiler-sfc**: Vue 文件解析
- **@babel/parser**: JavaScript/TypeScript 解析
- **@babel/traverse**: AST 遍历
- **fast-glob**: 文件扫描

#### 后端
- **@nestjs/sequelize**: Sequelize 集成
- **bull**: 队列管理
- **ioredis**: Redis 客户端
- **openai**: OpenAI API 客户端

#### 前端
- **vue-i18n**: Vue 国际化
- **element-plus**: UI 组件库
- **pinia**: 状态管理

### 8.2 性能优化

1. **增量提取**: 只提取变更的文件
2. **缓存机制**: Redis 缓存热点数据
3. **批量操作**: 批量上传、批量翻译
4. **CDN 加速**: 语言包托管到 CDN
5. **懒加载**: 前端按需加载语言包

### 8.3 安全性

1. **API Key 认证**: 每个项目独立的 API Key
2. **权限控制**: RBAC (admin/developer/translator)
3. **速率限制**: 防止 API 滥用
4. **数据加密**: 敏感信息加密存储

---

## 9. 部署方案

### 9.1 Docker Compose 部署

```yaml
version: '3.8'

services:
  # 前端
  web:
    build: ./apps/web
    ports:
      - "3000:80"
    depends_on:
      - api
  
  # 后端 API
  api:
    build: ./apps/api
    ports:
      - "3001:3000"
    environment:
      - DATABASE_URL=mysql://user:pass@db:3306/i18n
      - REDIS_URL=redis://redis:6379
      - OPENAI_API_KEY=${OPENAI_API_KEY}
    depends_on:
      - db
      - redis
  
  # 队列 Worker
  worker:
    build: ./apps/api
    command: npm run worker
    environment:
      - DATABASE_URL=mysql://user:pass@db:3306/i18n
      - REDIS_URL=redis://redis:6379
    depends_on:
      - db
      - redis
  
  # 数据库
  db:
    image: mysql:8.0
    environment:
      - MYSQL_ROOT_PASSWORD=rootpass
      - MYSQL_DATABASE=i18n
    volumes:
      - db-data:/var/lib/mysql
  
  # Redis
  redis:
    image: redis:7-alpine
    volumes:
      - redis-data:/data

volumes:
  db-data:
  redis-data:
```

### 9.2 环境变量配置

```env
# Database
DATABASE_URL=mysql://user:password@localhost:3306/i18n_platform

# Redis
REDIS_URL=redis://localhost:6379

# AI Services
OPENAI_API_KEY=sk-xxx
GOOGLE_TRANSLATE_KEY=xxx

# JWT
JWT_SECRET=your-secret-key

# AWS S3 (可选，用于存储导出文件)
AWS_ACCESS_KEY_ID=xxx
AWS_SECRET_ACCESS_KEY=xxx
AWS_S3_BUCKET=i18n-exports
```

---

## 10. 未来扩展

1. **更多框架支持**: Angular、Svelte 等
2. **机器学习优化**: 基于历史翻译优化 AI 翻译质量
3. **协作功能**: 翻译审核工作流、评论功能
4. **统计分析**: 翻译进度仪表盘、质量报告
5. **Webhook**: 翻译完成通知
6. **第三方集成**: Figma、Sketch 设计稿提取文案
7. **移动端支持**: iOS/Android 项目支持

---

## 11. 开发里程碑

### Phase 1: MVP (4-6 周)
- ✅ 基础项目结构搭建 (Turborepo)
- ✅ Vue 词条提取器
- ✅ CLI 基础命令 (extract, push, pull)
- ✅ 后端 API 核心接口
- ✅ 数据库设计与实现
- ✅ 前端基础页面

### Phase 2: AI 翻译 (2-3 周)
- ✅ 集成 OpenAI API
- ✅ 队列系统
- ✅ 批量翻译功能
- ✅ 翻译进度实时显示

### Phase 3: 版本控制 (2-3 周)
- ✅ 版本创建与管理
- ✅ 版本快照
- ✅ 版本对比功能

### Phase 4: 优化与上线 (2-3 周)
- ✅ React 提取器
- ✅ 性能优化
- ✅ 测试与 Bug 修复
- ✅ 文档完善
- ✅ 生产环境部署

---

## 12. 总结

本设计文档提供了一个完整的国际化管理平台解决方案，核心特性包括：

1. **自动化**: 从代码提取到翻译，尽可能自动化
2. **AI 赋能**: 利用 AI 大幅提升翻译效率
3. **版本控制**: 与开发流程紧密结合
4. **易用性**: 简单的 CLI 命令，直观的 Web 界面
5. **可扩展**: Monorepo 架构，易于扩展新功能

该平台将显著提升多语言项目的开发效率，降低翻译成本。
