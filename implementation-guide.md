# 国际化平台实现指南

## 1. 开发顺序建议

### 阶段 1：基础设施搭建 (Week 1)

#### 1.1 创建 Turborepo 项目
```bash
# 使用 pnpm 创建 monorepo
npx create-turbo@latest i18n-platform
cd i18n-platform

# 安装 pnpm (如果没有)
npm install -g pnpm

# 初始化
pnpm install
```

#### 1.2 创建包结构
```bash
# 创建目录
mkdir -p packages/{cli,parser,shared,sdk}
mkdir -p apps/{api,web}

# 初始化各个包
cd packages/shared && pnpm init
cd ../parser && pnpm init
cd ../sdk && pnpm init
cd ../cli && pnpm init
```

#### 1.3 配置 TypeScript
根目录 `tsconfig.json`:
```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "ESNext",
    "moduleResolution": "node",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "resolveJsonModule": true,
    "declaration": true,
    "declarationMap": true,
    "sourceMap": true,
    "composite": true
  }
}
```

### 阶段 2：共享包开发 (Week 1-2)

#### 2.1 packages/shared - 类型定义

```typescript
// packages/shared/src/types/entry.ts
export interface Entry {
  id?: number;
  key: string;
  sourceText: string;
  context?: string;
  filePath?: string;
  lineNumber?: number;
  namespace?: string;
  status?: EntryStatus;
}

export enum EntryStatus {
  PENDING = 'pending',
  TRANSLATING = 'translating',
  TRANSLATED = 'translated',
  APPROVED = 'approved',
}

// packages/shared/src/types/translation.ts
export interface Translation {
  id?: number;
  entryId: number;
  locale: string;
  translatedText: string;
  translationType: 'manual' | 'ai' | 'import';
  translatorId?: number;
  aiProvider?: string;
  qualityScore?: number;
  isReviewed: boolean;
}

// packages/shared/src/types/project.ts
export interface Project {
  id?: number;
  name: string;
  slug: string;
  description?: string;
  framework: Framework;
  defaultLocale: string;
  supportedLocales: string[];
  apiKey?: string;
}

export enum Framework {
  VUE = 'vue',
  REACT = 'react',
  ANGULAR = 'angular',
}

// packages/shared/src/types/version.ts
export interface Version {
  id?: number;
  projectId: number;
  versionName: string;
  branchName: string;
  commitHash?: string;
  isMaster: boolean;
  snapshot?: Record<string, any>;
  description?: string;
}
```

#### 2.2 packages/shared - 常量

```typescript
// packages/shared/src/constants/locales.ts
export const SUPPORTED_LOCALES = {
  'zh-CN': '简体中文',
  'zh-TW': '繁體中文',
  'en-US': 'English (US)',
  'en-GB': 'English (UK)',
  'ja-JP': '日本語',
  'ko-KR': '한국어',
  'de-DE': 'Deutsch',
  'fr-FR': 'Français',
  'es-ES': 'Español',
  'pt-BR': 'Português (Brasil)',
  'ru-RU': 'Русский',
  'ar-SA': 'العربية',
} as const;

export type LocaleCode = keyof typeof SUPPORTED_LOCALES;

// packages/shared/src/constants/frameworks.ts
export const FRAMEWORK_CONFIG = {
  vue: {
    extensions: ['.vue', '.js', '.ts'],
    functionNames: ['$t', 't', 'this.$t'],
    componentProps: ['i18n-t', 'v-t'],
  },
  react: {
    extensions: ['.jsx', '.tsx', '.js', '.ts'],
    functionNames: ['t', 'i18n.t'],
    components: ['Trans', 'Translation'],
  },
} as const;
```

### 阶段 3：解析器开发 (Week 2-3)

#### 3.1 packages/parser - Vue 解析器

```typescript
// packages/parser/src/extractors/vue.extractor.ts
import { parse, SFCDescriptor } from '@vue/compiler-sfc';
import * as babelParser from '@babel/parser';
import traverse from '@babel/traverse';
import { Entry } from '@i18n-platform/shared';

export class VueExtractor {
  private functionNames = ['$t', 't'];
  
  extract(fileContent: string, filePath: string): Entry[] {
    const entries: Entry[] = [];
    
    try {
      const { descriptor } = parse(fileContent, { filename: filePath });
      
      // 提取 template
      if (descriptor.template) {
        entries.push(...this.extractFromTemplate(
          descriptor.template.content,
          filePath
        ));
      }
      
      // 提取 script
      const scriptContent = descriptor.script?.content || 
                           descriptor.scriptSetup?.content;
      if (scriptContent) {
        entries.push(...this.extractFromScript(scriptContent, filePath));
      }
    } catch (error) {
      console.error(`Failed to parse ${filePath}:`, error);
    }
    
    return entries;
  }
  
  private extractFromTemplate(template: string, filePath: string): Entry[] {
    const entries: Entry[] = [];
    
    // 匹配 {{ $t('xxx') }} 或 v-t="'xxx'"
    const patterns = [
      /\{\{\s*\$t\(['"](.+?)['"]\)\s*\}\}/g,
      /v-t=['"](.+?)['"]/g,
      /:title="\$t\('(.+?)'\)"/g,
    ];
    
    patterns.forEach(pattern => {
      let match;
      while ((match = pattern.exec(template)) !== null) {
        entries.push({
          key: match[1],
          sourceText: match[1], // 初始值，后续可能需要查询实际文本
          filePath,
        });
      }
    });
    
    return entries;
  }
  
  private extractFromScript(code: string, filePath: string): Entry[] {
    const entries: Entry[] = [];
    
    try {
      const ast = babelParser.parse(code, {
        sourceType: 'module',
        plugins: ['typescript', 'jsx'],
      });
      
      traverse(ast, {
        CallExpression: (path) => {
          const { callee, arguments: args } = path.node;
          
          // 检查是否是 $t() 或 t() 调用
          const isI18nCall = 
            (callee.type === 'Identifier' && 
             this.functionNames.includes(callee.name)) ||
            (callee.type === 'MemberExpression' && 
             callee.property.type === 'Identifier' &&
             callee.property.name === 't');
          
          if (isI18nCall && args[0]?.type === 'StringLiteral') {
            const key = args[0].value;
            const lineNumber = path.node.loc?.start.line;
            
            entries.push({
              key,
              sourceText: key,
              filePath,
              lineNumber,
            });
          }
        },
      });
    } catch (error) {
      console.error(`Failed to parse script in ${filePath}:`, error);
    }
    
    return entries;
  }
}
```

#### 3.2 packages/parser - React 解析器

```typescript
// packages/parser/src/extractors/react.extractor.ts
import * as babelParser from '@babel/parser';
import traverse from '@babel/traverse';
import { Entry } from '@i18n-platform/shared';

export class ReactExtractor {
  extract(fileContent: string, filePath: string): Entry[] {
    const entries: Entry[] = [];
    
    try {
      const ast = babelParser.parse(fileContent, {
        sourceType: 'module',
        plugins: ['typescript', 'jsx'],
      });
      
      traverse(ast, {
        // 提取 t('xxx') 调用
        CallExpression: (path) => {
          const { callee, arguments: args } = path.node;
          
          if (
            callee.type === 'Identifier' && 
            callee.name === 't' &&
            args[0]?.type === 'StringLiteral'
          ) {
            entries.push({
              key: args[0].value,
              sourceText: args[0].value,
              filePath,
              lineNumber: path.node.loc?.start.line,
            });
          }
        },
        
        // 提取 <Trans i18nKey="xxx" /> 组件
        JSXElement: (path) => {
          const openingElement = path.node.openingElement;
          
          if (
            openingElement.name.type === 'JSXIdentifier' &&
            openingElement.name.name === 'Trans'
          ) {
            const i18nKeyAttr = openingElement.attributes.find(
              attr => 
                attr.type === 'JSXAttribute' &&
                attr.name.name === 'i18nKey'
            );
            
            if (
              i18nKeyAttr?.type === 'JSXAttribute' &&
              i18nKeyAttr.value?.type === 'StringLiteral'
            ) {
              entries.push({
                key: i18nKeyAttr.value.value,
                sourceText: i18nKeyAttr.value.value,
                filePath,
                lineNumber: path.node.loc?.start.line,
              });
            }
          }
        },
      });
    } catch (error) {
      console.error(`Failed to parse ${filePath}:`, error);
    }
    
    return entries;
  }
}
```

### 阶段 4：CLI 工具开发 (Week 3-4)

#### 4.1 packages/cli - 核心命令

```typescript
// packages/cli/src/commands/extract.ts
import { Command } from 'commander';
import { VueExtractor } from '@i18n-platform/parser';
import { ReactExtractor } from '@i18n-platform/parser';
import { loadConfig } from '../config/loader';
import fg from 'fast-glob';
import fs from 'fs-extra';
import ora from 'ora';
import chalk from 'chalk';

export const extractCommand = new Command('extract')
  .description('Extract i18n entries from source code')
  .option('-c, --config <path>', 'Config file path', './i18n.config.js')
  .option('-o, --output <path>', 'Output directory')
  .action(async (options) => {
    const spinner = ora('Loading configuration...').start();
    
    try {
      // 加载配置
      const config = await loadConfig(options.config);
      spinner.succeed('Configuration loaded');
      
      // 选择提取器
      const Extractor = config.framework === 'vue' 
        ? VueExtractor 
        : ReactExtractor;
      const extractor = new Extractor();
      
      // 扫描文件
      spinner.start('Scanning files...');
      const files = await fg(config.extract.include, {
        ignore: config.extract.exclude,
        absolute: true,
      });
      spinner.succeed(`Found ${files.length} files`);
      
      // 提取词条
      spinner.start('Extracting entries...');
      const allEntries = [];
      
      for (const file of files) {
        const content = await fs.readFile(file, 'utf-8');
        const entries = extractor.extract(content, file);
        allEntries.push(...entries);
      }
      
      // 去重
      const uniqueEntries = deduplicateEntries(allEntries);
      spinner.succeed(`Extracted ${uniqueEntries.length} unique entries`);
      
      // 输出
      const outputDir = options.output || config.extract.outputDir;
      await fs.ensureDir(outputDir);
      const outputPath = `${outputDir}/extracted.json`;
      await fs.writeJSON(outputPath, uniqueEntries, { spaces: 2 });
      
      console.log(chalk.green(`\n✅ Extraction complete!`));
      console.log(chalk.gray(`Output: ${outputPath}`));
      
    } catch (error) {
      spinner.fail('Extraction failed');
      console.error(chalk.red(error.message));
      process.exit(1);
    }
  });

function deduplicateEntries(entries: any[]) {
  const map = new Map();
  
  entries.forEach(entry => {
    if (!map.has(entry.key)) {
      map.set(entry.key, entry);
    }
  });
  
  return Array.from(map.values());
}
```

```typescript
// packages/cli/src/commands/push.ts
import { Command } from 'commander';
import { I18nApiClient } from '@i18n-platform/sdk';
import { loadConfig } from '../config/loader';
import fs from 'fs-extra';
import ora from 'ora';
import chalk from 'chalk';

export const pushCommand = new Command('push')
  .description('Push extracted entries to platform')
  .option('-f, --file <path>', 'Entries file path')
  .action(async (options) => {
    const spinner = ora('Loading configuration...').start();
    
    try {
      const config = await loadConfig();
      const client = new I18nApiClient({
        apiKey: config.apiKey,
        endpoint: config.apiEndpoint,
      });
      
      // 读取词条文件
      const filePath = options.file || `${config.extract.outputDir}/extracted.json`;
      const entries = await fs.readJSON(filePath);
      
      spinner.text = `Uploading ${entries.length} entries...`;
      
      // 上传
      const result = await client.entries.batchCreate(
        config.projectId,
        entries,
        {
          branch: config.version.branch,
        }
      );
      
      spinner.succeed('Upload complete!');
      
      console.log(chalk.green('\n✅ Push successful!'));
      console.log(chalk.gray(`  Created: ${result.created}`));
      console.log(chalk.gray(`  Updated: ${result.updated}`));
      
      if (result.errors?.length > 0) {
        console.log(chalk.yellow(`\n⚠️  ${result.errors.length} errors:`));
        result.errors.forEach(err => {
          console.log(chalk.red(`  - ${err.key}: ${err.error}`));
        });
      }
      
    } catch (error) {
      spinner.fail('Push failed');
      console.error(chalk.red(error.message));
      process.exit(1);
    }
  });
```

```typescript
// packages/cli/src/commands/pull.ts
import { Command } from 'commander';
import { I18nApiClient } from '@i18n-platform/sdk';
import { loadConfig } from '../config/loader';
import fs from 'fs-extra';
import ora from 'ora';
import chalk from 'chalk';
import path from 'path';

export const pullCommand = new Command('pull')
  .description('Pull translations from platform')
  .option('-l, --locales <locales>', 'Locales to pull (comma-separated)')
  .option('--version <version>', 'Version to pull')
  .action(async (options) => {
    const config = await loadConfig();
    const client = new I18nApiClient({
      apiKey: config.apiKey,
      endpoint: config.apiEndpoint,
    });
    
    const locales = options.locales?.split(',') || config.targetLocales;
    const version = options.version || 'master';
    
    console.log(chalk.blue(`Pulling translations for ${locales.join(', ')}...\n`));
    
    for (const locale of locales) {
      const spinner = ora(`Pulling ${locale}...`).start();
      
      try {
        // 拉取翻译
        const translations = await client.translations.pull(
          config.projectId,
          locale,
          { version }
        );
        
        // 写入文件
        const filename = config.output.filename.replace('[locale]', locale);
        const outputPath = path.join(config.output.localesDir, filename);
        
        await fs.ensureDir(path.dirname(outputPath));
        
        if (config.output.format === 'json') {
          await fs.writeJSON(outputPath, translations, { spaces: 2 });
        } else if (config.output.format === 'yaml') {
          const yaml = require('js-yaml');
          await fs.writeFile(outputPath, yaml.dump(translations));
        }
        
        spinner.succeed(`${locale} pulled successfully`);
        console.log(chalk.gray(`  → ${outputPath}`));
        
      } catch (error) {
        spinner.fail(`Failed to pull ${locale}`);
        console.error(chalk.red(`  ${error.message}`));
      }
    }
    
    console.log(chalk.green('\n✅ Pull complete!'));
  });
```

#### 4.2 packages/sdk - API 客户端

```typescript
// packages/sdk/src/client.ts
import axios, { AxiosInstance } from 'axios';

export interface I18nApiClientConfig {
  apiKey: string;
  endpoint: string;
  timeout?: number;
}

export class I18nApiClient {
  private http: AxiosInstance;
  
  public entries: EntriesApi;
  public translations: TranslationsApi;
  public versions: VersionsApi;
  
  constructor(config: I18nApiClientConfig) {
    this.http = axios.create({
      baseURL: config.endpoint,
      timeout: config.timeout || 30000,
      headers: {
        'X-API-Key': config.apiKey,
        'Content-Type': 'application/json',
      },
    });
    
    this.entries = new EntriesApi(this.http);
    this.translations = new TranslationsApi(this.http);
    this.versions = new VersionsApi(this.http);
  }
}

// packages/sdk/src/api/entry.api.ts
import { AxiosInstance } from 'axios';
import { Entry } from '@i18n-platform/shared';

export class EntriesApi {
  constructor(private http: AxiosInstance) {}
  
  async batchCreate(
    projectId: string,
    entries: Entry[],
    options?: { branch?: string }
  ) {
    const response = await this.http.post(
      `/projects/${projectId}/entries/batch`,
      { entries, ...options }
    );
    return response.data;
  }
  
  async findAll(projectId: string, query?: any) {
    const response = await this.http.get(
      `/projects/${projectId}/entries`,
      { params: query }
    );
    return response.data;
  }
}
```

### 阶段 5：后端 API 开发 (Week 4-6)

#### 5.1 apps/api - 项目搭建

```bash
cd apps/api
npm install @nestjs/cli -g
nest new . --skip-git

# 安装依赖
pnpm add @nestjs/sequelize sequelize mysql2
pnpm add @nestjs/config @nestjs/jwt @nestjs/passport passport passport-jwt
pnpm add bull @nestjs/bull ioredis
pnpm add openai
pnpm add class-validator class-transformer
pnpm add -D @types/passport-jwt sequelize-cli
```

#### 5.2 数据库模型

```typescript
// apps/api/src/modules/entry/entities/entry.entity.ts
import {
  Table,
  Column,
  Model,
  DataType,
  ForeignKey,
  BelongsTo,
  HasMany,
} from 'sequelize-typescript';
import { Project } from '../../project/entities/project.entity';
import { Translation } from '../../translation/entities/translation.entity';

@Table({
  tableName: 'entries',
  timestamps: true,
  underscored: true,
})
export class Entry extends Model {
  @Column({
    type: DataType.BIGINT,
    primaryKey: true,
    autoIncrement: true,
  })
  id: number;
  
  @ForeignKey(() => Project)
  @Column({
    type: DataType.BIGINT,
    allowNull: false,
  })
  projectId: number;
  
  @Column({
    type: DataType.STRING(255),
    allowNull: false,
  })
  entryKey: string;
  
  @Column({
    type: DataType.TEXT,
    allowNull: false,
  })
  sourceText: string;
  
  @Column({
    type: DataType.STRING(500),
  })
  context: string;
  
  @Column({
    type: DataType.STRING(500),
  })
  filePath: string;
  
  @Column({
    type: DataType.INTEGER,
  })
  lineNumber: number;
  
  @Column({
    type: DataType.STRING(100),
    defaultValue: 'common',
  })
  namespace: string;
  
  @Column({
    type: DataType.ENUM('pending', 'translating', 'translated', 'approved'),
    defaultValue: 'pending',
  })
  status: string;
  
  @BelongsTo(() => Project)
  project: Project;
  
  @HasMany(() => Translation)
  translations: Translation[];
}
```

#### 5.3 核心服务

```typescript
// apps/api/src/modules/entry/entry.service.ts
import { Injectable } from '@nestjs/common';
import { InjectModel } from '@nestjs/sequelize';
import { Entry } from './entities/entry.entity';
import { Translation } from '../translation/entities/translation.entity';
import { BatchCreateEntriesDto } from './dto/batch-create-entries.dto';

@Injectable()
export class EntryService {
  constructor(
    @InjectModel(Entry)
    private entryModel: typeof Entry,
  ) {}
  
  async batchCreateOrUpdate(
    projectId: number,
    dto: BatchCreateEntriesDto,
  ) {
    const results = {
      created: 0,
      updated: 0,
      errors: [],
    };
    
    for (const entryData of dto.entries) {
      try {
        const [entry, created] = await this.entryModel.findOrCreate({
          where: {
            projectId,
            entryKey: entryData.key,
          },
          defaults: {
            sourceText: entryData.sourceText || entryData.key,
            context: entryData.context,
            filePath: entryData.filePath,
            lineNumber: entryData.lineNumber,
            namespace: entryData.namespace || 'common',
          },
        });
        
        if (!created) {
          // 检查源文本是否变化
          if (entry.sourceText !== entryData.sourceText) {
            await entry.update({
              sourceText: entryData.sourceText,
              filePath: entryData.filePath,
              lineNumber: entryData.lineNumber,
            });
            
            // 标记相关翻译需要重新审核
            await Translation.update(
              { isReviewed: false },
              { where: { entryId: entry.id } }
            );
            
            results.updated++;
          }
        } else {
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
  
  async findAll(projectId: number, query: any) {
    const { page = 1, limit = 20, keyword, status, locale } = query;
    const offset = (page - 1) * limit;
    
    const where: any = { projectId };
    
    if (keyword) {
      where[Op.or] = [
        { entryKey: { [Op.like]: `%${keyword}%` } },
        { sourceText: { [Op.like]: `%${keyword}%` } },
      ];
    }
    
    if (status) {
      where.status = status;
    }
    
    const include = [];
    
    if (locale) {
      include.push({
        model: Translation,
        where: { locale },
        required: false,
      });
    }
    
    const { rows, count } = await this.entryModel.findAndCountAll({
      where,
      include,
      limit,
      offset,
      order: [['createdAt', 'DESC']],
    });
    
    return {
      data: rows,
      total: count,
      page,
      limit,
    };
  }
}
```

#### 5.4 AI 翻译服务

```typescript
// apps/api/src/modules/ai/providers/openai.service.ts
import { Injectable } from '@nestjs/common';
import { ConfigService } from '@nestjs/config';
import OpenAI from 'openai';

@Injectable()
export class OpenAiService {
  private openai: OpenAI;
  
  constructor(private configService: ConfigService) {
    this.openai = new OpenAI({
      apiKey: this.configService.get('OPENAI_API_KEY'),
    });
  }
  
  async translateBatch(
    texts: string[],
    sourceLang: string,
    targetLang: string,
  ): Promise<string[]> {
    const prompt = `You are a professional translator. Translate the following texts from ${sourceLang} to ${targetLang}.

IMPORTANT RULES:
1. Keep all placeholders intact: {variable}, {{interpolation}}, %s, %d, etc.
2. Preserve HTML tags: <br>, <b>, <span>, etc.
3. Maintain the same format and structure
4. Return ONLY a JSON array of translated strings, no explanations

Texts to translate:
${JSON.stringify(texts, null, 2)}

Return format: ["translation1", "translation2", ...]`;
    
    try {
      const response = await this.openai.chat.completions.create({
        model: 'gpt-4',
        messages: [{ role: 'user', content: prompt }],
        temperature: 0.3,
        max_tokens: 4000,
      });
      
      const content = response.choices[0].message.content;
      const translations = JSON.parse(content);
      
      if (!Array.isArray(translations) || translations.length !== texts.length) {
        throw new Error('Invalid translation response format');
      }
      
      return translations;
    } catch (error) {
      console.error('OpenAI translation error:', error);
      throw new Error(`Translation failed: ${error.message}`);
    }
  }
  
  async translateSingle(
    text: string,
    sourceLang: string,
    targetLang: string,
  ): Promise<string> {
    const [translation] = await this.translateBatch([text], sourceLang, targetLang);
    return translation;
  }
}
```

```typescript
// apps/api/src/modules/translation/translation.processor.ts
import { Processor, Process } from '@nestjs/bull';
import { Job } from 'bull';
import { Injectable } from '@nestjs/common';
import { InjectModel } from '@nestjs/sequelize';
import { TranslationJob } from './entities/translation-job.entity';
import { Entry } from '../entry/entities/entry.entity';
import { Translation } from './entities/translation.entity';
import { OpenAiService } from '../ai/providers/openai.service';

@Processor('translation')
@Injectable()
export class TranslationProcessor {
  constructor(
    @InjectModel(TranslationJob)
    private jobModel: typeof TranslationJob,
    @InjectModel(Entry)
    private entryModel: typeof Entry,
    @InjectModel(Translation)
    private translationModel: typeof Translation,
    private openaiService: OpenAiService,
  ) {}
  
  @Process('ai-translate')
  async handleTranslation(job: Job) {
    const { jobId, projectId, sourceLocale, targetLocale, entryIds } = job.data;
    
    try {
      // 更新任务状态
      await this.jobModel.update(
        { status: 'processing', startedAt: new Date() },
        { where: { id: jobId } }
      );
      
      // 获取待翻译的词条
      const where: any = { projectId };
      if (entryIds?.length > 0) {
        where.id = entryIds;
      }
      
      const entries = await this.entryModel.findAll({
        where,
        include: [{
          model: Translation,
          where: { locale: targetLocale },
          required: false,
        }],
      });
      
      // 过滤出没有翻译的词条
      const entriesToTranslate = entries.filter(
        entry => !entry.translations?.length
      );
      
      const total = entriesToTranslate.length;
      let translated = 0;
      
      // 分批翻译（每批 20 条）
      const BATCH_SIZE = 20;
      
      for (let i = 0; i < entriesToTranslate.length; i += BATCH_SIZE) {
        const batch = entriesToTranslate.slice(i, i + BATCH_SIZE);
        
        // 调用 AI 翻译
        const sourceTexts = batch.map(e => e.sourceText);
        const translations = await this.openaiService.translateBatch(
          sourceTexts,
          sourceLocale,
          targetLocale,
        );
        
        // 保存翻译结果
        await Promise.all(
          batch.map((entry, index) =>
            this.translationModel.create({
              entryId: entry.id,
              locale: targetLocale,
              translatedText: translations[index],
              translationType: 'ai',
              aiProvider: 'openai',
              isReviewed: false,
            })
          )
        );
        
        translated += batch.length;
        
        // 更新进度
        await this.jobModel.update(
          { translatedEntries: translated },
          { where: { id: jobId } }
        );
        
        // 报告进度
        await job.progress((translated / total) * 100);
      }
      
      // 完成
      await this.jobModel.update(
        { status: 'completed', completedAt: new Date() },
        { where: { id: jobId } }
      );
      
      return { success: true, translated };
      
    } catch (error) {
      await this.jobModel.update(
        {
          status: 'failed',
          errorMessage: error.message,
          completedAt: new Date(),
        },
        { where: { id: jobId } }
      );
      
      throw error;
    }
  }
}
```

### 阶段 6：前端开发 (Week 6-8)

#### 6.1 apps/web - 项目搭建

```bash
cd apps/web
pnpm create vite . --template vue-ts
pnpm add vue-router pinia
pnpm add element-plus
pnpm add axios
pnpm add @vueuse/core
```

#### 6.2 核心组件示例

```vue
<!-- apps/web/src/views/Entries/EntryList.vue -->
<template>
  <div class="entry-list">
    <el-card>
      <template #header>
        <div class="header">
          <h2>词条管理</h2>
          <el-button type="primary" @click="handleBatchTranslate">
            批量 AI 翻译
          </el-button>
        </div>
      </template>
      
      <div class="toolbar">
        <el-input
          v-model="searchKeyword"
          placeholder="搜索词条..."
          style="width: 300px"
          @input="handleSearch"
        >
          <template #prefix>
            <el-icon><Search /></el-icon>
          </template>
        </el-input>
        
        <el-select v-model="statusFilter" placeholder="状态筛选">
          <el-option label="全部" value="" />
          <el-option label="待翻译" value="pending" />
          <el-option label="已翻译" value="translated" />
          <el-option label="已审核" value="approved" />
        </el-select>
      </div>
      
      <el-table
        :data="entries"
        v-loading="loading"
        @selection-change="handleSelectionChange"
      >
        <el-table-column type="selection" width="55" />
        
        <el-table-column prop="entryKey" label="Key" min-width="200">
          <template #default="{ row }">
            <el-tag size="small">{{ row.namespace }}</el-tag>
            <span class="ml-2">{{ row.entryKey }}</span>
          </template>
        </el-table-column>
        
        <el-table-column prop="sourceText" label="源文本" min-width="200" />
        
        <el-table-column label="翻译进度" width="150">
          <template #default="{ row }">
            <el-progress
              :percentage="getTranslationProgress(row)"
              :color="getProgressColor(row)"
            />
          </template>
        </el-table-column>
        
        <el-table-column label="状态" width="100">
          <template #default="{ row }">
            <el-tag :type="getStatusType(row.status)">
              {{ getStatusText(row.status) }}
            </el-tag>
          </template>
        </el-table-column>
        
        <el-table-column label="操作" width="150" fixed="right">
          <template #default="{ row }">
            <el-button link type="primary" @click="handleEdit(row)">
              编辑
            </el-button>
            <el-button link type="danger" @click="handleDelete(row)">
              删除
            </el-button>
          </template>
        </el-table-column>
      </el-table>
      
      <el-pagination
        v-model:current-page="page"
        v-model:page-size="limit"
        :total="total"
        layout="total, prev, pager, next, sizes"
        @current-change="fetchEntries"
        @size-change="fetchEntries"
      />
    </el-card>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue';
import { useRoute } from 'vue-router';
import { ElMessage } from 'element-plus';
import { entryApi } from '@/api/entry';

const route = useRoute();
const projectId = route.params.projectId as string;

const entries = ref([]);
const loading = ref(false);
const searchKeyword = ref('');
const statusFilter = ref('');
const selectedEntries = ref([]);
const page = ref(1);
const limit = ref(20);
const total = ref(0);

const fetchEntries = async () => {
  loading.value = true;
  try {
    const { data, total: t } = await entryApi.findAll(projectId, {
      page: page.value,
      limit: limit.value,
      keyword: searchKeyword.value,
      status: statusFilter.value,
    });
    entries.value = data;
    total.value = t;
  } catch (error) {
    ElMessage.error('加载失败');
  } finally {
    loading.value = false;
  }
};

const handleBatchTranslate = () => {
  // 实现批量翻译逻辑
};

onMounted(() => {
  fetchEntries();
});
</script>
```

### 阶段 7：测试与优化 (Week 9-10)

#### 7.1 单元测试

```typescript
// packages/parser/test/vue.extractor.spec.ts
import { describe, it, expect } from 'vitest';
import { VueExtractor } from '../src/extractors/vue.extractor';

describe('VueExtractor', () => {
  const extractor = new VueExtractor();
  
  it('should extract from template', () => {
    const code = `
      <template>
        <div>{{ $t('home.welcome') }}</div>
      </template>
    `;
    
    const entries = extractor.extract(code, 'test.vue');
    expect(entries).toHaveLength(1);
    expect(entries[0].key).toBe('home.welcome');
  });
  
  it('should extract from script', () => {
    const code = `
      <script setup>
      const message = $t('home.title');
      </script>
    `;
    
    const entries = extractor.extract(code, 'test.vue');
    expect(entries).toHaveLength(1);
    expect(entries[0].key).toBe('home.title');
  });
});
```

#### 7.2 性能优化

1. **数据库索引优化**
```sql
-- 为常用查询添加索引
CREATE INDEX idx_entry_project_status ON entries(project_id, status);
CREATE INDEX idx_translation_entry_locale ON translations(entry_id, locale);
CREATE INDEX idx_version_project_master ON versions(project_id, is_master);
```

2. **Redis 缓存**
```typescript
// 缓存项目配置
async getProject(id: number) {
  const cacheKey = `project:${id}`;
  const cached = await this.redis.get(cacheKey);
  
  if (cached) {
    return JSON.parse(cached);
  }
  
  const project = await this.projectModel.findByPk(id);
  await this.redis.setex(cacheKey, 3600, JSON.stringify(project));
  
  return project;
}
```

3. **批量操作优化**
```typescript
// 使用事务批量插入
async batchInsert(entries: Entry[]) {
  const transaction = await this.sequelize.transaction();
  
  try {
    await this.entryModel.bulkCreate(entries, { transaction });
    await transaction.commit();
  } catch (error) {
    await transaction.rollback();
    throw error;
  }
}
```

## 2. 关键技术点

### 2.1 AST 解析优化

使用增量解析，只解析变更的文件：

```typescript
// 保存文件 hash，检测变化
import crypto from 'crypto';

function getFileHash(content: string): string {
  return crypto.createHash('md5').update(content).digest('hex');
}

async function extractIncremental(files: string[]) {
  const cache = await loadCache(); // 加载上次的 hash 缓存
  const changed = [];
  
  for (const file of files) {
    const content = await fs.readFile(file, 'utf-8');
    const hash = getFileHash(content);
    
    if (cache[file] !== hash) {
      changed.push(file);
      cache[file] = hash;
    }
  }
  
  await saveCache(cache);
  return changed; // 只提取变更的文件
}
```

### 2.2 队列重试机制

```typescript
// 配置 Bull 队列重试
const queue = new Bull('translation', {
  redis: redisConfig,
  defaultJobOptions: {
    attempts: 3,
    backoff: {
      type: 'exponential',
      delay: 2000,
    },
    removeOnComplete: true,
    removeOnFail: false,
  },
});

// 监听失败事件
queue.on('failed', (job, error) => {
  console.error(`Job ${job.id} failed:`, error);
  // 发送通知或记录日志
});
```

### 2.3 版本快照优化

使用 JSON 存储快照，避免创建大量关联记录：

```typescript
async createVersionSnapshot(projectId: number, versionName: string) {
  // 查询所有词条和翻译
  const entries = await this.entryModel.findAll({
    where: { projectId },
    include: [{ model: Translation }],
  });
  
  // 构建快照
  const snapshot = {};
  entries.forEach(entry => {
    snapshot[entry.entryKey] = {
      sourceText: entry.sourceText,
      translations: entry.translations.reduce((acc, t) => {
        acc[t.locale] = t.translatedText;
        return acc;
      }, {}),
    };
  });
  
  // 保存版本
  await this.versionModel.create({
    projectId,
    versionName,
    snapshot: JSON.stringify(snapshot),
    isMaster: false,
  });
}
```

## 3. 总结

完整的实现步骤：
1. ✅ 搭建 Turborepo monorepo
2. ✅ 开发共享类型和工具包
3. ✅ 实现 Vue/React 解析器
4. ✅ 开发 CLI 工具
5. ✅ 构建后端 API (NestJS)
6. ✅ 集成 AI 翻译
7. ✅ 开发前端管理平台
8. ✅ 实现版本管理
9. ✅ 测试与优化
10. ✅ 部署上线

关键成功因素：
- 模块化设计，职责清晰
- 使用成熟的开源库，避免重复造轮子
- 性能优化：缓存、批量、索引
- 良好的错误处理和重试机制
- 完善的文档和示例
