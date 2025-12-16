# Turborepo é¡¹ç›®ç»“æ„è¯¦ç»†è¯´æ˜

## å®Œæ•´ç›®å½•ç»“æ„

```
translatex/
â”œâ”€â”€ .github/
â”?  â””â”€â”€ workflows/
â”?      â”œâ”€â”€ ci.yml                    # CI/CD é…ç½®
â”?      â””â”€â”€ release.yml               # å‘å¸ƒæµç¨‹
â”?
â”œâ”€â”€ apps/
â”?  â”œâ”€â”€ web/                          # å‰ç«¯ç®¡ç†å¹³å°
â”?  â”?  â”œâ”€â”€ public/
â”?  â”?  â”œâ”€â”€ src/
â”?  â”?  â”?  â”œâ”€â”€ assets/              # é™æ€èµ„æº?
â”?  â”?  â”?  â”œâ”€â”€ components/          # å…¬å…±ç»„ä»¶
â”?  â”?  â”?  â”?  â”œâ”€â”€ EntryTable/
â”?  â”?  â”?  â”?  â”œâ”€â”€ TranslationEditor/
â”?  â”?  â”?  â”?  â””â”€â”€ VersionSelector/
â”?  â”?  â”?  â”œâ”€â”€ views/               # é¡µé¢è§†å›¾
â”?  â”?  â”?  â”?  â”œâ”€â”€ Dashboard.vue
â”?  â”?  â”?  â”?  â”œâ”€â”€ Projects/
â”?  â”?  â”?  â”?  â”?  â”œâ”€â”€ ProjectList.vue
â”?  â”?  â”?  â”?  â”?  â”œâ”€â”€ ProjectDetail.vue
â”?  â”?  â”?  â”?  â”?  â””â”€â”€ ProjectSettings.vue
â”?  â”?  â”?  â”?  â”œâ”€â”€ Entries/
â”?  â”?  â”?  â”?  â”?  â”œâ”€â”€ EntryList.vue
â”?  â”?  â”?  â”?  â”?  â””â”€â”€ EntryEditor.vue
â”?  â”?  â”?  â”?  â”œâ”€â”€ Translations/
â”?  â”?  â”?  â”?  â”?  â”œâ”€â”€ TranslationBoard.vue
â”?  â”?  â”?  â”?  â”?  â””â”€â”€ BatchTranslate.vue
â”?  â”?  â”?  â”?  â””â”€â”€ Versions/
â”?  â”?  â”?  â”?      â”œâ”€â”€ VersionList.vue
â”?  â”?  â”?  â”?      â””â”€â”€ VersionCompare.vue
â”?  â”?  â”?  â”œâ”€â”€ router/              # è·¯ç”±é…ç½®
â”?  â”?  â”?  â”œâ”€â”€ store/               # Pinia Store
â”?  â”?  â”?  â”?  â”œâ”€â”€ project.ts
â”?  â”?  â”?  â”?  â”œâ”€â”€ entry.ts
â”?  â”?  â”?  â”?  â””â”€â”€ translation.ts
â”?  â”?  â”?  â”œâ”€â”€ api/                 # API è°ƒç”¨
â”?  â”?  â”?  â”?  â”œâ”€â”€ client.ts
â”?  â”?  â”?  â”?  â”œâ”€â”€ project.ts
â”?  â”?  â”?  â”?  â”œâ”€â”€ entry.ts
â”?  â”?  â”?  â”?  â””â”€â”€ translation.ts
â”?  â”?  â”?  â”œâ”€â”€ utils/
â”?  â”?  â”?  â”œâ”€â”€ App.vue
â”?  â”?  â”?  â””â”€â”€ main.ts
â”?  â”?  â”œâ”€â”€ index.html
â”?  â”?  â”œâ”€â”€ vite.config.ts
â”?  â”?  â”œâ”€â”€ package.json
â”?  â”?  â””â”€â”€ tsconfig.json
â”?  â”?
â”?  â””â”€â”€ api/                          # åç«¯ API æœåŠ¡
â”?      â”œâ”€â”€ src/
â”?      â”?  â”œâ”€â”€ common/              # å…¬å…±æ¨¡å—
â”?      â”?  â”?  â”œâ”€â”€ decorators/
â”?      â”?  â”?  â”œâ”€â”€ filters/
â”?      â”?  â”?  â”œâ”€â”€ guards/
â”?      â”?  â”?  â”?  â””â”€â”€ api-key.guard.ts
â”?      â”?  â”?  â”œâ”€â”€ interceptors/
â”?      â”?  â”?  â””â”€â”€ pipes/
â”?      â”?  â”œâ”€â”€ config/              # é…ç½®
â”?      â”?  â”?  â”œâ”€â”€ database.config.ts
â”?      â”?  â”?  â”œâ”€â”€ redis.config.ts
â”?      â”?  â”?  â””â”€â”€ ai.config.ts
â”?      â”?  â”œâ”€â”€ modules/
â”?      â”?  â”?  â”œâ”€â”€ auth/            # è®¤è¯æ¨¡å—
â”?      â”?  â”?  â”?  â”œâ”€â”€ auth.controller.ts
â”?      â”?  â”?  â”?  â”œâ”€â”€ auth.service.ts
â”?      â”?  â”?  â”?  â”œâ”€â”€ auth.module.ts
â”?      â”?  â”?  â”?  â””â”€â”€ strategies/
â”?      â”?  â”?  â”?      â”œâ”€â”€ jwt.strategy.ts
â”?      â”?  â”?  â”?      â””â”€â”€ api-key.strategy.ts
â”?      â”?  â”?  â”œâ”€â”€ user/            # ç”¨æˆ·æ¨¡å—
â”?      â”?  â”?  â”?  â”œâ”€â”€ user.controller.ts
â”?      â”?  â”?  â”?  â”œâ”€â”€ user.service.ts
â”?      â”?  â”?  â”?  â”œâ”€â”€ user.module.ts
â”?      â”?  â”?  â”?  â”œâ”€â”€ entities/
â”?      â”?  â”?  â”?  â”?  â””â”€â”€ user.entity.ts
â”?      â”?  â”?  â”?  â””â”€â”€ dto/
â”?      â”?  â”?  â”œâ”€â”€ project/         # é¡¹ç›®æ¨¡å—
â”?      â”?  â”?  â”?  â”œâ”€â”€ project.controller.ts
â”?      â”?  â”?  â”?  â”œâ”€â”€ project.service.ts
â”?      â”?  â”?  â”?  â”œâ”€â”€ project.module.ts
â”?      â”?  â”?  â”?  â”œâ”€â”€ entities/
â”?      â”?  â”?  â”?  â”?  â””â”€â”€ project.entity.ts
â”?      â”?  â”?  â”?  â””â”€â”€ dto/
â”?      â”?  â”?  â”?      â”œâ”€â”€ create-project.dto.ts
â”?      â”?  â”?  â”?      â””â”€â”€ update-project.dto.ts
â”?      â”?  â”?  â”œâ”€â”€ entry/           # è¯æ¡æ¨¡å—
â”?      â”?  â”?  â”?  â”œâ”€â”€ entry.controller.ts
â”?      â”?  â”?  â”?  â”œâ”€â”€ entry.service.ts
â”?      â”?  â”?  â”?  â”œâ”€â”€ entry.module.ts
â”?      â”?  â”?  â”?  â”œâ”€â”€ entities/
â”?      â”?  â”?  â”?  â”?  â””â”€â”€ entry.entity.ts
â”?      â”?  â”?  â”?  â””â”€â”€ dto/
â”?      â”?  â”?  â”?      â”œâ”€â”€ batch-create-entries.dto.ts
â”?      â”?  â”?  â”?      â””â”€â”€ find-entries.dto.ts
â”?      â”?  â”?  â”œâ”€â”€ translation/     # ç¿»è¯‘æ¨¡å—
â”?      â”?  â”?  â”?  â”œâ”€â”€ translation.controller.ts
â”?      â”?  â”?  â”?  â”œâ”€â”€ translation.service.ts
â”?      â”?  â”?  â”?  â”œâ”€â”€ translation.module.ts
â”?      â”?  â”?  â”?  â”œâ”€â”€ entities/
â”?      â”?  â”?  â”?  â”?  â”œâ”€â”€ translation.entity.ts
â”?      â”?  â”?  â”?  â”?  â””â”€â”€ translation-job.entity.ts
â”?      â”?  â”?  â”?  â””â”€â”€ dto/
â”?      â”?  â”?  â”œâ”€â”€ version/         # ç‰ˆæœ¬æ¨¡å—
â”?      â”?  â”?  â”?  â”œâ”€â”€ version.controller.ts
â”?      â”?  â”?  â”?  â”œâ”€â”€ version.service.ts
â”?      â”?  â”?  â”?  â”œâ”€â”€ version.module.ts
â”?      â”?  â”?  â”?  â”œâ”€â”€ entities/
â”?      â”?  â”?  â”?  â”?  â”œâ”€â”€ version.entity.ts
â”?      â”?  â”?  â”?  â”?  â””â”€â”€ entry-version.entity.ts
â”?      â”?  â”?  â”?  â””â”€â”€ dto/
â”?      â”?  â”?  â”œâ”€â”€ ai/              # AI ç¿»è¯‘æ¨¡å—
â”?      â”?  â”?  â”?  â”œâ”€â”€ ai.controller.ts
â”?      â”?  â”?  â”?  â”œâ”€â”€ ai.service.ts
â”?      â”?  â”?  â”?  â”œâ”€â”€ ai.module.ts
â”?      â”?  â”?  â”?  â””â”€â”€ providers/
â”?      â”?  â”?  â”?      â”œâ”€â”€ openai.service.ts
â”?      â”?  â”?  â”?      â”œâ”€â”€ google-translate.service.ts
â”?      â”?  â”?  â”?      â””â”€â”€ deepl.service.ts
â”?      â”?  â”?  â”œâ”€â”€ queue/           # é˜Ÿåˆ—æ¨¡å—
â”?      â”?  â”?  â”?  â”œâ”€â”€ queue.module.ts
â”?      â”?  â”?  â”?  â”œâ”€â”€ processors/
â”?      â”?  â”?  â”?  â”?  â””â”€â”€ translation.processor.ts
â”?      â”?  â”?  â”?  â””â”€â”€ producers/
â”?      â”?  â”?  â””â”€â”€ export/          # å¯¼å‡ºæ¨¡å—
â”?      â”?  â”?      â”œâ”€â”€ export.controller.ts
â”?      â”?  â”?      â”œâ”€â”€ export.service.ts
â”?      â”?  â”?      â””â”€â”€ export.module.ts
â”?      â”?  â”œâ”€â”€ database/
â”?      â”?  â”?  â””â”€â”€ migrations/      # æ•°æ®åº“è¿ç§?
â”?      â”?  â”œâ”€â”€ app.module.ts
â”?      â”?  â”œâ”€â”€ main.ts
â”?      â”?  â””â”€â”€ worker.ts            # Queue Worker å…¥å£
â”?      â”œâ”€â”€ test/
â”?      â”œâ”€â”€ package.json
â”?      â”œâ”€â”€ tsconfig.json
â”?      â””â”€â”€ nest-cli.json
â”?
â”œâ”€â”€ packages/
â”?  â”œâ”€â”€ cli/                          # CLI å·¥å…·åŒ?
â”?  â”?  â”œâ”€â”€ bin/
â”?  â”?  â”?  â””â”€â”€ i18n-cli.js          # å¯æ‰§è¡Œæ–‡ä»?
â”?  â”?  â”œâ”€â”€ src/
â”?  â”?  â”?  â”œâ”€â”€ commands/
â”?  â”?  â”?  â”?  â”œâ”€â”€ init.ts          # åˆå§‹åŒ–å‘½ä»?
â”?  â”?  â”?  â”?  â”œâ”€â”€ extract.ts       # æå–å‘½ä»¤
â”?  â”?  â”?  â”?  â”œâ”€â”€ push.ts          # ä¸Šä¼ å‘½ä»¤
â”?  â”?  â”?  â”?  â”œâ”€â”€ pull.ts          # ä¸‹è½½å‘½ä»¤
â”?  â”?  â”?  â”?  â”œâ”€â”€ status.ts        # çŠ¶æ€å‘½ä»?
â”?  â”?  â”?  â”?  â””â”€â”€ version/
â”?  â”?  â”?  â”?      â”œâ”€â”€ create.ts
â”?  â”?  â”?  â”?      â””â”€â”€ master.ts
â”?  â”?  â”?  â”œâ”€â”€ config/
â”?  â”?  â”?  â”?  â”œâ”€â”€ loader.ts        # é…ç½®åŠ è½½å™?
â”?  â”?  â”?  â”?  â””â”€â”€ schema.ts        # é…ç½®éªŒè¯
â”?  â”?  â”?  â”œâ”€â”€ utils/
â”?  â”?  â”?  â”?  â”œâ”€â”€ logger.ts
â”?  â”?  â”?  â”?  â”œâ”€â”€ file-scanner.ts
â”?  â”?  â”?  â”?  â””â”€â”€ spinner.ts
â”?  â”?  â”?  â”œâ”€â”€ index.ts
â”?  â”?  â”?  â””â”€â”€ cli.ts
â”?  â”?  â”œâ”€â”€ templates/               # é…ç½®æ¨¡æ¿
â”?  â”?  â”?  â””â”€â”€ i18n.config.template.js
â”?  â”?  â”œâ”€â”€ package.json
â”?  â”?  â””â”€â”€ tsconfig.json
â”?  â”?
â”?  â”œâ”€â”€ parser/                       # è¯æ¡è§£æå™¨æ ¸å¿?
â”?  â”?  â”œâ”€â”€ src/
â”?  â”?  â”?  â”œâ”€â”€ extractors/
â”?  â”?  â”?  â”?  â”œâ”€â”€ base.extractor.ts
â”?  â”?  â”?  â”?  â”œâ”€â”€ vue.extractor.ts
â”?  â”?  â”?  â”?  â””â”€â”€ react.extractor.ts
â”?  â”?  â”?  â”œâ”€â”€ ast/
â”?  â”?  â”?  â”?  â”œâ”€â”€ vue-ast-parser.ts
â”?  â”?  â”?  â”?  â”œâ”€â”€ jsx-ast-parser.ts
â”?  â”?  â”?  â”?  â””â”€â”€ template-parser.ts
â”?  â”?  â”?  â”œâ”€â”€ utils/
â”?  â”?  â”?  â”?  â”œâ”€â”€ deduplicate.ts
â”?  â”?  â”?  â”?  â””â”€â”€ normalize.ts
â”?  â”?  â”?  â”œâ”€â”€ types/
â”?  â”?  â”?  â”?  â””â”€â”€ entry.ts
â”?  â”?  â”?  â””â”€â”€ index.ts
â”?  â”?  â”œâ”€â”€ test/
â”?  â”?  â”?  â””â”€â”€ fixtures/            # æµ‹è¯•ç”¨ä¾‹
â”?  â”?  â”œâ”€â”€ package.json
â”?  â”?  â””â”€â”€ tsconfig.json
â”?  â”?
â”?  â”œâ”€â”€ shared/                       # å…±äº«ä»£ç 
â”?  â”?  â”œâ”€â”€ src/
â”?  â”?  â”?  â”œâ”€â”€ types/
â”?  â”?  â”?  â”?  â”œâ”€â”€ project.ts
â”?  â”?  â”?  â”?  â”œâ”€â”€ entry.ts
â”?  â”?  â”?  â”?  â”œâ”€â”€ translation.ts
â”?  â”?  â”?  â”?  â”œâ”€â”€ version.ts
â”?  â”?  â”?  â”?  â””â”€â”€ api.ts
â”?  â”?  â”?  â”œâ”€â”€ constants/
â”?  â”?  â”?  â”?  â”œâ”€â”€ locales.ts       # æ”¯æŒçš„è¯­è¨€åˆ—è¡¨
â”?  â”?  â”?  â”?  â”œâ”€â”€ status.ts
â”?  â”?  â”?  â”?  â””â”€â”€ frameworks.ts
â”?  â”?  â”?  â”œâ”€â”€ utils/
â”?  â”?  â”?  â”?  â”œâ”€â”€ date.ts
â”?  â”?  â”?  â”?  â”œâ”€â”€ string.ts
â”?  â”?  â”?  â”?  â””â”€â”€ validation.ts
â”?  â”?  â”?  â”œâ”€â”€ validators/          # é€šç”¨éªŒè¯å™?
â”?  â”?  â”?  â””â”€â”€ index.ts
â”?  â”?  â”œâ”€â”€ package.json
â”?  â”?  â””â”€â”€ tsconfig.json
â”?  â”?
â”?  â””â”€â”€ sdk/                          # API SDK (ä¾?CLI ä½¿ç”¨)
â”?      â”œâ”€â”€ src/
â”?      â”?  â”œâ”€â”€ client.ts            # HTTP å®¢æˆ·ç«?
â”?      â”?  â”œâ”€â”€ api/
â”?      â”?  â”?  â”œâ”€â”€ project.api.ts
â”?      â”?  â”?  â”œâ”€â”€ entry.api.ts
â”?      â”?  â”?  â”œâ”€â”€ translation.api.ts
â”?      â”?  â”?  â””â”€â”€ version.api.ts
â”?      â”?  â”œâ”€â”€ types/
â”?      â”?  â””â”€â”€ index.ts
â”?      â”œâ”€â”€ package.json
â”?      â””â”€â”€ tsconfig.json
â”?
â”œâ”€â”€ docs/                             # æ–‡æ¡£
â”?  â”œâ”€â”€ getting-started.md
â”?  â”œâ”€â”€ cli-usage.md
â”?  â”œâ”€â”€ api-reference.md
â”?  â””â”€â”€ deployment.md
â”?
â”œâ”€â”€ .gitignore
â”œâ”€â”€ .eslintrc.js
â”œâ”€â”€ .prettierrc
â”œâ”€â”€ package.json                      # æ ?package.json
â”œâ”€â”€ pnpm-workspace.yaml              # pnpm workspace é…ç½®
â”œâ”€â”€ turbo.json                        # Turborepo é…ç½®
â”œâ”€â”€ tsconfig.json                     # æ ?TypeScript é…ç½®
â””â”€â”€ README.md
```

## package.json ç¤ºä¾‹

### æ ?package.json

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
  "name": "@translatex/cli",
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
    "@translatex/parser": "workspace:*",
    "@translatex/sdk": "workspace:*",
    "@translatex/shared": "workspace:*",
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
  "name": "@translatex/parser",
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

## turbo.json é…ç½®

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

## ä¾èµ–å…³ç³»å›?

```
apps/web
  â”œâ”€â†?@translatex/shared
  â””â”€â†?@translatex/sdk

apps/api
  â”œâ”€â†?@translatex/shared
  â””â”€â†?@translatex/parser (ç”¨äºéªŒè¯æå–ç»“æœ)

packages/cli
  â”œâ”€â†?@translatex/parser
  â”œâ”€â†?@translatex/sdk
  â””â”€â†?@translatex/shared

packages/sdk
  â””â”€â†?@translatex/shared

packages/parser
  â””â”€â†?@translatex/shared (types)

packages/shared
  (æ— ä¾èµ?
```

## å¼€å‘å·¥ä½œæµ

### 1. å®‰è£…ä¾èµ–
```bash
pnpm install
```

### 2. å¼€å‘æ¨¡å¼?
```bash
# å¯åŠ¨æ‰€æœ‰æœåŠ?
pnpm dev

# æˆ–å•ç‹¬å¯åŠ?
pnpm --filter @translatex/web dev
pnpm --filter @translatex/api dev
```

### 3. æ„å»º
```bash
# æ„å»ºæ‰€æœ‰åŒ…
pnpm build

# æ„å»ºç‰¹å®šåŒ?
pnpm --filter @translatex/cli build
```

### 4. æµ‹è¯•
```bash
pnpm test
```

### 5. å‘å¸ƒ CLI
```bash
cd packages/cli
pnpm publish
```

## ç¯å¢ƒå˜é‡ç®¡ç†

```
i18n-platform/
â”œâ”€â”€ .env.example              # ç¤ºä¾‹ç¯å¢ƒå˜é‡
â”œâ”€â”€ apps/
â”?  â”œâ”€â”€ web/
â”?  â”?  â”œâ”€â”€ .env.development
â”?  â”?  â””â”€â”€ .env.production
â”?  â””â”€â”€ api/
â”?      â”œâ”€â”€ .env.development
â”?      â””â”€â”€ .env.production
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
