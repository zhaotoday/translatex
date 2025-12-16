# 实现指南

## 开发计划 (10周)

| 阶段 | 时间 | 任务 |
|------|------|------|
| Phase 1 | Week 1-2 | 基础设施 + 共享包 |
| Phase 2 | Week 3-4 | 解析器 + CLI 工具 |
| Phase 3 | Week 4-6 | 后端 API + AI 翻译 |
| Phase 4 | Week 6-8 | 前端管理平台 |
| Phase 5 | Week 9-10 | 测试 + 优化 + 上线 |

## Phase 1: 基础设施

### 1.1 创建 Turborepo 项目

```bash
npx create-turbo@latest translatex
cd translatex
pnpm install
```

### 1.2 创建包结构

```bash
mkdir -p packages/{cli,parser,shared,sdk}
mkdir -p apps/{api,web}
```

### 1.3 共享类型 (packages/shared)

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
  isReviewed: boolean;
}

// packages/shared/src/constants/locales.ts
export const SUPPORTED_LOCALES = {
  'zh-CN': '简体中文',
  'en-US': 'English (US)',
  'ja-JP': '日本語',
  'ko-KR': '한국어',
} as const;
```

## Phase 2: 解析器 + CLI

### 2.1 Vue 解析器 (packages/parser)

```typescript
// packages/parser/src/extractors/vue.extractor.ts
import { parse } from '@vue/compiler-sfc';
import * as babelParser from '@babel/parser';
import traverse from '@babel/traverse';
import { Entry } from '@translatex/shared';

export class VueExtractor {
  extract(fileContent: string, filePath: string): Entry[] {
    const entries: Entry[] = [];
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
    
    return entries;
  }
  
  private extractFromTemplate(template: string, filePath: string): Entry[] {
    const entries: Entry[] = [];
    const patterns = [
      /\{\{\s*\$t\(['"](.+?)['"]\)\s*\}\}/g,
      /v-t=['"](.+?)['"]/g,
    ];
    
    patterns.forEach(pattern => {
      let match;
      while ((match = pattern.exec(template)) !== null) {
        entries.push({
          key: match[1],
          sourceText: match[1],
          filePath,
        });
      }
    });
    
    return entries;
  }
  
  private extractFromScript(code: string, filePath: string): Entry[] {
    const entries: Entry[] = [];
    const ast = babelParser.parse(code, {
      sourceType: 'module',
      plugins: ['typescript', 'jsx'],
    });
    
    traverse(ast, {
      CallExpression: (path) => {
        const { callee, arguments: args } = path.node;
        
        if (
          (callee.type === 'Identifier' && callee.name === '$t') ||
          (callee.type === 'MemberExpression' && 
           callee.property.name === 't')
        ) {
          if (args[0]?.type === 'StringLiteral') {
            entries.push({
              key: args[0].value,
              sourceText: args[0].value,
              filePath,
              lineNumber: path.node.loc?.start.line,
            });
          }
        }
      },
    });
    
    return entries;
  }
}
```

### 2.2 CLI 命令 (packages/cli)

```typescript
// packages/cli/src/commands/extract.ts
import { Command } from 'commander';
import { VueExtractor } from '@translatex/parser';
import { loadConfig } from '../config/loader';
import fg from 'fast-glob';
import fs from 'fs-extra';
import ora from 'ora';
import chalk from 'chalk';

export const extractCommand = new Command('extract')
  .description('Extract i18n entries from source code')
  .action(async () => {
    const spinner = ora('Loading configuration...').start();
    
    const config = await loadConfig();
    spinner.succeed('Configuration loaded');
    
    const extractor = new VueExtractor();
    
    spinner.start('Scanning files...');
    const files = await fg(config.extract.include, {
      ignore: config.extract.exclude,
      absolute: true,
    });
    spinner.succeed(`Found ${files.length} files`);
    
    spinner.start('Extracting entries...');
    const allEntries = [];
    
    for (const file of files) {
      const content = await fs.readFile(file, 'utf-8');
      const entries = extractor.extract(content, file);
      allEntries.push(...entries);
    }
    
    const uniqueEntries = deduplicateEntries(allEntries);
    spinner.succeed(`Extracted ${uniqueEntries.length} entries`);
    
    const outputPath = `${config.extract.outputDir}/extracted.json`;
    await fs.ensureDir(config.extract.outputDir);
    await fs.writeJSON(outputPath, uniqueEntries, { spaces: 2 });
    
    console.log(chalk.green(`\n✓ Extraction complete!`));
    console.log(chalk.gray(`Output: ${outputPath}`));
  });

function deduplicateEntries(entries: Entry[]): Entry[] {
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
import { I18nApiClient } from '@translatex/sdk';
import { loadConfig } from '../config/loader';
import fs from 'fs-extra';
import ora from 'ora';
import chalk from 'chalk';

export const pushCommand = new Command('push')
  .description('Push entries to platform')
  .action(async () => {
    const config = await loadConfig();
    const client = new I18nApiClient({
      apiKey: config.apiKey,
      endpoint: config.apiEndpoint,
    });
    
    const filePath = `${config.extract.outputDir}/extracted.json`;
    const entries = await fs.readJSON(filePath);
    
    const spinner = ora(`Uploading ${entries.length} entries...`).start();
    
    const result = await client.entries.batchCreate(
      config.projectId,
      entries,
      { branch: config.version.branch }
    );
    
    spinner.succeed('Upload complete!');
    
    console.log(chalk.green('\n✓ Push successful!'));
    console.log(chalk.gray(`  Created: ${result.created}`));
    console.log(chalk.gray(`  Updated: ${result.updated}`));
  });
```

```typescript
// packages/cli/src/commands/pull.ts
import { Command } from 'commander';
import { I18nApiClient } from '@translatex/sdk';
import { loadConfig } from '../config/loader';
import fs from 'fs-extra';
import ora from 'ora';
import chalk from 'chalk';
import path from 'path';

export const pullCommand = new Command('pull')
  .description('Pull translations from platform')
  .option('-l, --locales <locales>', 'Locales to pull (comma-separated)')
  .action(async (options) => {
    const config = await loadConfig();
    const client = new I18nApiClient({
      apiKey: config.apiKey,
      endpoint: config.apiEndpoint,
    });
    
    const locales = options.locales?.split(',') || config.targetLocales;
    
    console.log(chalk.blue(`Pulling ${locales.join(', ')}...\n`));
    
    for (const locale of locales) {
      const spinner = ora(`Pulling ${locale}...`).start();
      
      const translations = await client.translations.pull(
        config.projectId,
        locale,
        { version: 'master' }
      );
      
      const filename = config.output.filename.replace('[locale]', locale);
      const outputPath = path.join(config.output.localesDir, filename);
      
      await fs.ensureDir(path.dirname(outputPath));
      await fs.writeJSON(outputPath, translations, { spaces: 2 });
      
      spinner.succeed(`${locale} pulled successfully`);
      console.log(chalk.gray(`  → ${outputPath}`));
    }
  });
```

## Phase 3: 后端 API

### 3.1 Entry 服务 (apps/api)

```typescript
// apps/api/src/modules/entry/entry.service.ts
import { Injectable } from '@nestjs/common';
import { InjectModel } from '@nestjs/sequelize';
import { Entry } from './entities/entry.entity';

@Injectable()
export class EntryService {
  constructor(
    @InjectModel(Entry)
    private entryModel: typeof Entry,
  ) {}
  
  async batchCreateOrUpdate(projectId: number, entries: Entry[]) {
    const results = { created: 0, updated: 0, errors: [] };
    
    for (const entryData of entries) {
      try {
        const [entry, created] = await this.entryModel.findOrCreate({
          where: {
            projectId,
            entryKey: entryData.key,
          },
          defaults: {
            sourceText: entryData.sourceText,
            filePath: entryData.filePath,
            lineNumber: entryData.lineNumber,
          },
        });
        
        if (created) {
          results.created++;
        } else if (entry.sourceText !== entryData.sourceText) {
          await entry.update({
            sourceText: entryData.sourceText,
          });
          results.updated++;
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

### 3.2 AI 翻译服务

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
    const prompt = `You are a professional translator. 
Translate the following texts from ${sourceLang} to ${targetLang}.

IMPORTANT: Keep placeholders like {variable}, {{interpolation}}, %s intact.
Return ONLY a JSON array of translated strings.

Texts:
${JSON.stringify(texts, null, 2)}`;
    
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

### 3.3 翻译队列处理器

```typescript
// apps/api/src/modules/translation/translation.processor.ts
import { Processor, Process } from '@nestjs/bull';
import { Job } from 'bull';
import { InjectModel } from '@nestjs/sequelize';
import { Entry } from '../entry/entities/entry.entity';
import { Translation } from './entities/translation.entity';
import { OpenAiService } from '../ai/providers/openai.service';

@Processor('translation')
export class TranslationProcessor {
  constructor(
    @InjectModel(Entry)
    private entryModel: typeof Entry,
    @InjectModel(Translation)
    private translationModel: typeof Translation,
    private openaiService: OpenAiService,
  ) {}
  
  @Process('ai-translate')
  async handleTranslation(job: Job) {
    const { projectId, sourceLocale, targetLocale } = job.data;
    
    const entries = await this.entryModel.findAll({
      where: { projectId },
    });
    
    const BATCH_SIZE = 20;
    let translated = 0;
    
    for (let i = 0; i < entries.length; i += BATCH_SIZE) {
      const batch = entries.slice(i, i + BATCH_SIZE);
      const sourceTexts = batch.map(e => e.sourceText);
      
      const translations = await this.openaiService.translateBatch(
        sourceTexts,
        sourceLocale,
        targetLocale,
      );
      
      await Promise.all(
        batch.map((entry, index) =>
          this.translationModel.create({
            entryId: entry.id,
            locale: targetLocale,
            translatedText: translations[index],
            translationType: 'ai',
            aiProvider: 'openai',
          })
        )
      );
      
      translated += batch.length;
      await job.progress((translated / entries.length) * 100);
    }
    
    return { success: true, translated };
  }
}
```

## Phase 4: 前端

### 4.1 词条列表组件

```vue
<!-- apps/web/src/views/Entries/EntryList.vue -->
<template>
  <el-card>
    <template #header>
      <div class="header">
        <h2>词条管理</h2>
        <el-button type="primary" @click="handleBatchTranslate">
          AI 翻译
        </el-button>
      </div>
    </template>
    
    <el-input
      v-model="searchKeyword"
      placeholder="搜索词条..."
      style="width: 300px; margin-bottom: 20px"
    />
    
    <el-table :data="entries" v-loading="loading">
      <el-table-column prop="entryKey" label="Key" />
      <el-table-column prop="sourceText" label="源文本" />
      <el-table-column label="翻译进度">
        <template #default="{ row }">
          <el-progress :percentage="getProgress(row)" />
        </template>
      </el-table-column>
      <el-table-column label="操作">
        <template #default="{ row }">
          <el-button link @click="handleEdit(row)">编辑</el-button>
        </template>
      </el-table-column>
    </el-table>
  </el-card>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue';
import { entryApi } from '@/api/entry';

const entries = ref([]);
const loading = ref(false);
const searchKeyword = ref('');

const fetchEntries = async () => {
  loading.value = true;
  try {
    const { data } = await entryApi.findAll({
      keyword: searchKeyword.value,
    });
    entries.value = data;
  } finally {
    loading.value = false;
  }
};

onMounted(() => {
  fetchEntries();
});
</script>
```

## Phase 5: 测试 + 优化

### 5.1 单元测试

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
});
```

### 5.2 性能优化

```typescript
// Redis 缓存
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

```sql
-- 数据库索引
CREATE INDEX idx_entry_project ON entries(project_id, status);
CREATE INDEX idx_translation_entry_locale ON translations(entry_id, locale);
```

## 总结

完整实现步骤：
1. ✓ 搭建 Turborepo monorepo
2. ✓ 开发共享类型和工具包
3. ✓ 实现 Vue/React 解析器
4. ✓ 开发 CLI 工具
5. ✓ 构建后端 API (NestJS)
6. ✓ 集成 AI 翻译
7. ✓ 开发前端管理平台
8. ✓ 实现版本管理
9. ✓ 测试与优化
10. ✓ 部署上线
