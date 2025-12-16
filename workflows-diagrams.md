# 国际化平台流程图

## 1. 整体系统架构流程

```mermaid
graph TB
    subgraph "开发者本地环境"
        A[前端项目<br/>Vue/React] 
        B[CLI 工具<br/>@i18n-platform/cli]
        C[配置文件<br/>i18n.config.js]
        D[语言包目录<br/>locales/]
    end
    
    subgraph "i18n 管理平台"
        E[Nginx<br/>负载均衡]
        F[前端应用<br/>Vue 3]
        G[后端 API<br/>NestJS]
        H[Queue Worker<br/>后台任务]
        I[MySQL<br/>数据库]
        J[Redis<br/>缓存/队列]
        K[AI 服务<br/>OpenAI/Google]
    end
    
    A -->|extract| B
    B -->|读取配置| C
    B -->|push 词条| E
    E -->|HTTPS| G
    G -->|存储| I
    G -->|缓存| J
    
    F -->|API 调用| G
    G -->|发布任务| J
    J -->|消费任务| H
    H -->|调用| K
    H -->|保存结果| I
    
    B -->|pull 翻译| E
    E -->|返回语言包| B
    B -->|写入| D
    A -->|加载| D
    
    style A fill:#e1f5ff
    style B fill:#fff4e1
    style F fill:#e8f5e9
    style G fill:#f3e5f5
    style K fill:#ffe0b2
```

## 2. CLI 工作流程

### 2.1 初始化流程

```mermaid
sequenceDiagram
    participant Dev as 开发者
    participant CLI as i18n-cli
    participant API as 平台 API
    participant FS as 文件系统
    
    Dev->>CLI: i18n-cli init
    CLI->>Dev: 请输入项目信息
    Dev-->>CLI: 项目名称、框架等
    CLI->>API: POST /api/projects<br/>创建项目
    API-->>CLI: 返回 projectId 和 apiKey
    CLI->>FS: 创建 i18n.config.js
    FS-->>CLI: 配置文件已创建
    CLI->>Dev: ✅ 初始化完成
```

### 2.2 提取词条流程

```mermaid
flowchart TD
    Start([i18n-cli extract]) --> LoadConfig[加载配置文件]
    LoadConfig --> ScanFiles[扫描源代码文件<br/>根据 include/exclude]
    
    ScanFiles --> CheckFramework{检查框架类型}
    CheckFramework -->|Vue| VueParser[Vue 解析器]
    CheckFramework -->|React| ReactParser[React 解析器]
    
    VueParser --> ParseTemplate[解析 Template<br/>查找 $t 调用]
    VueParser --> ParseScript[解析 Script<br/>AST 遍历]
    
    ReactParser --> ParseJSX[解析 JSX<br/>查找 t() 和 Trans]
    
    ParseTemplate --> Collect[收集所有词条]
    ParseScript --> Collect
    ParseJSX --> Collect
    
    Collect --> Deduplicate[去重处理]
    Deduplicate --> AddMeta[添加元信息<br/>文件路径、行号]
    AddMeta --> Output[输出到 JSON 文件<br/>locales/extracted/]
    
    Output --> End([✅ 提取完成])
    
    style Start fill:#4CAF50,color:#fff
    style End fill:#4CAF50,color:#fff
    style VueParser fill:#42b983
    style ReactParser fill:#61dafb
```

### 2.3 上传词条流程

```mermaid
sequenceDiagram
    participant Dev as 开发者
    participant CLI as i18n-cli
    participant FS as 文件系统
    participant API as 平台 API
    participant DB as 数据库
    
    Dev->>CLI: i18n-cli push
    CLI->>FS: 读取 extracted.json
    FS-->>CLI: 词条数据
    
    CLI->>CLI: 验证词条格式
    CLI->>API: POST /api/projects/:id/entries/batch<br/>Header: X-API-Key
    
    API->>API: 验证 API Key
    API->>DB: 查询现有词条
    
    loop 遍历每个词条
        API->>DB: 检查词条是否存在
        alt 词条不存在
            API->>DB: INSERT 新词条
        else 词条已存在
            alt 源文本有变化
                API->>DB: UPDATE 词条
                API->>DB: 标记相关翻译为待审核
            end
        end
    end
    
    DB-->>API: 操作结果
    API-->>CLI: {created: 10, updated: 5}
    CLI->>Dev: ✅ 上传成功<br/>新增 10 条，更新 5 条
```

### 2.4 下载翻译流程

```mermaid
sequenceDiagram
    participant Dev as 开发者
    participant CLI as i18n-cli
    participant API as 平台 API
    participant DB as 数据库
    participant FS as 文件系统
    
    Dev->>CLI: i18n-cli pull --locales en-US,ja-JP
    CLI->>CLI: 读取配置<br/>获取 projectId、apiKey
    
    loop 每个语言
        CLI->>API: GET /api/projects/:id/export/:locale<br/>?version=master
        API->>DB: 查询主版本
        API->>DB: 获取该语言所有翻译
        
        DB-->>API: 翻译数据
        API->>API: 组装成语言包<br/>{key: value} 格式
        
        alt format=json
            API-->>CLI: JSON 格式
        else format=yaml
            API-->>CLI: YAML 格式
        end
        
        CLI->>FS: 写入文件<br/>src/locales/en-US.json
    end
    
    FS-->>CLI: 文件已保存
    CLI->>Dev: ✅ 下载完成<br/>en-US.json, ja-JP.json
```

## 3. 平台翻译流程

### 3.1 手动翻译流程

```mermaid
stateDiagram-v2
    [*] --> 待翻译: 词条上传
    
    待翻译 --> 翻译中: 翻译者认领
    翻译中 --> 已翻译: 提交翻译
    
    已翻译 --> 审核中: 提交审核
    审核中 --> 已通过: 审核通过
    审核中 --> 翻译中: 审核拒绝<br/>(需修改)
    
    已通过 --> [*]
    
    note right of 待翻译
        status: pending
    end note
    
    note right of 已翻译
        status: translated
        is_reviewed: false
    end note
    
    note right of 已通过
        status: approved
        is_reviewed: true
    end note
```

### 3.2 AI 翻译流程

```mermaid
flowchart TD
    Start([用户点击<br/>批量 AI 翻译]) --> SelectLocales[选择目标语言<br/>en-US, ja-JP]
    SelectLocales --> CreateJobs[创建翻译任务<br/>translation_jobs]
    
    CreateJobs --> Job1[Job 1: zh-CN → en-US]
    CreateJobs --> Job2[Job 2: zh-CN → ja-JP]
    
    Job1 --> Queue1[加入 Redis 队列]
    Job2 --> Queue2[加入 Redis 队列]
    
    Queue1 --> Worker1[Worker 消费任务]
    Queue2 --> Worker2[Worker 消费任务]
    
    Worker1 --> UpdateStatus1[更新状态: processing]
    Worker2 --> UpdateStatus2[更新状态: processing]
    
    UpdateStatus1 --> GetEntries1[查询待翻译词条<br/>status != translated]
    UpdateStatus2 --> GetEntries2[查询待翻译词条]
    
    GetEntries1 --> Batch1[分批处理<br/>每批 20 条]
    GetEntries2 --> Batch2[分批处理]
    
    Batch1 --> CallAI1[调用 OpenAI API]
    Batch2 --> CallAI2[调用 OpenAI API]
    
    CallAI1 --> SaveResult1[保存翻译结果<br/>translation_type: ai]
    CallAI2 --> SaveResult2[保存翻译结果]
    
    SaveResult1 --> CheckComplete1{是否完成?}
    SaveResult2 --> CheckComplete2{是否完成?}
    
    CheckComplete1 -->|否| Batch1
    CheckComplete1 -->|是| Complete1[状态: completed]
    
    CheckComplete2 -->|否| Batch2
    CheckComplete2 -->|是| Complete2[状态: completed]
    
    Complete1 --> Notify1[WebSocket 通知前端]
    Complete2 --> Notify2[WebSocket 通知前端]
    
    Notify1 --> End([任务完成])
    Notify2 --> End
    
    style Start fill:#2196F3,color:#fff
    style End fill:#4CAF50,color:#fff
    style CallAI1 fill:#FF9800
    style CallAI2 fill:#FF9800
```

### 3.3 AI 翻译详细序列图

```mermaid
sequenceDiagram
    participant User as 用户
    participant Web as 前端
    participant API as API 服务
    participant Queue as Redis 队列
    participant Worker as Queue Worker
    participant OpenAI as OpenAI API
    participant DB as 数据库
    
    User->>Web: 点击"批量 AI 翻译"
    Web->>API: POST /api/projects/:id/translations/ai-translate<br/>{targetLocales: ['en-US', 'ja-JP']}
    
    API->>DB: 创建 translation_jobs 记录
    DB-->>API: job_ids: [1, 2]
    
    API->>Queue: publish('translation', {jobId: 1, ...})
    API->>Queue: publish('translation', {jobId: 2, ...})
    
    API-->>Web: {jobs: [{id: 1, status: 'pending'}, ...]}
    Web->>User: 显示翻译任务列表
    
    Queue->>Worker: consume task (jobId: 1)
    Worker->>DB: UPDATE job SET status='processing'
    Worker->>DB: 通过 WebSocket 通知前端
    Web->>User: 更新 UI：翻译中...
    
    Worker->>DB: SELECT entries WHERE project_id=? AND NOT EXISTS translation
    DB-->>Worker: [entry1, entry2, ..., entry100]
    
    loop 分批处理 (每批 20 条)
        Worker->>OpenAI: POST /chat/completions<br/>{texts: [...], target: 'en-US'}
        OpenAI-->>Worker: [translation1, translation2, ...]
        
        Worker->>DB: INSERT INTO translations<br/>(entry_id, locale, text, type='ai')
        
        Worker->>DB: UPDATE job SET translated_entries=20
        Worker->>Web: WebSocket 推送进度 (20/100)
        Web->>User: 更新进度条：20%
    end
    
    Worker->>DB: UPDATE job SET status='completed'
    Worker->>Web: WebSocket 通知完成
    Web->>User: ✅ 翻译完成！
```

## 4. 版本管理流程

### 4.1 版本创建流程

```mermaid
flowchart TD
    Start([开发者创建版本]) --> CLI[i18n-cli version:create v1.0.0]
    
    CLI --> GetBranch[获取当前 Git 分支]
    GetBranch --> GetCommit[获取最新 commit hash]
    
    GetCommit --> CallAPI[POST /api/projects/:id/versions<br/>{versionName, branchName, commitHash}]
    
    CallAPI --> QueryEntries[查询当前所有词条]
    QueryEntries --> QueryTranslations[查询所有翻译]
    
    QueryTranslations --> CreateSnapshot[创建快照<br/>保存词条和翻译状态]
    
    CreateSnapshot --> InsertVersion[INSERT INTO versions<br/>{..., snapshot: {...}}]
    
    InsertVersion --> CreateAssociations[创建词条版本关联<br/>entry_versions 表]
    
    CreateAssociations --> Response[返回版本信息]
    Response --> CLI
    
    CLI --> Success([✅ 版本创建成功<br/>v1.0.0])
    
    style Start fill:#2196F3,color:#fff
    style Success fill:#4CAF50,color:#fff
```

### 4.2 设置主版本流程

```mermaid
sequenceDiagram
    participant Dev as 开发者
    participant CLI as CLI
    participant API as API
    participant DB as 数据库
    
    Dev->>CLI: i18n-cli version:master v1.0.0
    CLI->>API: POST /api/versions/:id/set-master
    
    API->>DB: BEGIN TRANSACTION
    
    API->>DB: UPDATE versions<br/>SET is_master=false<br/>WHERE project_id=? AND is_master=true
    
    API->>DB: UPDATE versions<br/>SET is_master=true<br/>WHERE id=? AND version_name='v1.0.0'
    
    API->>DB: COMMIT
    
    DB-->>API: Success
    API-->>CLI: {success: true, message: 'v1.0.0 is now master'}
    CLI->>Dev: ✅ v1.0.0 已设置为主版本
```

### 4.3 版本对比流程

```mermaid
flowchart LR
    subgraph "版本 v1.0.0"
        V1E1[home.welcome: 欢迎]
        V1E2[home.title: 首页]
        V1E3[user.name: 用户名]
    end
    
    subgraph "版本 v1.1.0"
        V2E1[home.welcome: 欢迎使用]
        V2E2[home.title: 首页]
        V2E3[user.name: 用户名]
        V2E4[user.email: 邮箱]
    end
    
    V1E1 -.更新.-> V2E1
    V1E2 -.无变化.-> V2E2
    V1E3 -.无变化.-> V2E3
    V2E4[新增词条]
    
    style V2E1 fill:#FFF3E0
    style V2E4 fill:#E8F5E9
```

## 5. 数据流转图

### 5.1 词条生命周期

```mermaid
graph LR
    A[源代码] -->|extract| B[extracted.json]
    B -->|push| C[平台数据库<br/>entries 表]
    C -->|AI/手动翻译| D[translations 表]
    D -->|创建版本| E[entry_versions 表<br/>快照]
    E -->|pull| F[本地语言包<br/>locales/*.json]
    F -->|加载| G[前端应用<br/>运行时]
    
    style A fill:#e3f2fd
    style C fill:#fff3e0
    style D fill:#e8f5e9
    style E fill:#f3e5f5
    style G fill:#fce4ec
```

### 5.2 完整的用户旅程

```mermaid
journey
    title 国际化开发者旅程
    section 项目初始化
      安装 CLI: 5: 开发者
      运行 init: 5: 开发者
      配置项目: 4: 开发者
    
    section 开发阶段
      编写代码使用 $t(): 5: 开发者
      运行 extract: 5: 开发者
      推送词条: 5: 开发者
    
    section 翻译阶段
      查看词条: 4: 翻译人员
      AI 翻译: 5: 翻译人员
      人工审核: 4: 翻译人员
      标记完成: 5: 翻译人员
    
    section 发布阶段
      拉取翻译: 5: 开发者
      创建版本: 5: 开发者
      设为主版本: 5: 开发者
      部署上线: 5: DevOps
```

## 6. API 调用流程

### 6.1 认证流程

```mermaid
sequenceDiagram
    participant CLI as CLI 工具
    participant Guard as ApiKeyGuard
    participant DB as 数据库
    participant Controller as Controller
    
    CLI->>Guard: Request + Header(X-API-Key)
    Guard->>Guard: 提取 API Key
    Guard->>DB: SELECT project WHERE api_key=?
    
    alt API Key 有效
        DB-->>Guard: Project 信息
        Guard->>Guard: 注入 project 到 request
        Guard->>Controller: 继续处理请求
        Controller-->>CLI: 正常响应
    else API Key 无效
        DB-->>Guard: null
        Guard-->>CLI: 401 Unauthorized
    end
```

### 6.2 批量导出流程

```mermaid
flowchart TD
    Start([GET /api/projects/:id/export/all]) --> Auth[验证 API Key]
    Auth --> GetVersion[获取版本信息<br/>默认 master]
    
    GetVersion --> QueryData[查询该版本的<br/>所有词条和翻译]
    
    QueryData --> Loop{遍历每个语言}
    
    Loop -->|en-US| BuildEN[构建 en-US.json<br/>{key: translation}]
    Loop -->|ja-JP| BuildJP[构建 ja-JP.json]
    Loop -->|zh-CN| BuildCN[构建 zh-CN.json]
    
    BuildEN --> AddToZip[添加到 ZIP]
    BuildJP --> AddToZip
    BuildCN --> AddToZip
    
    AddToZip --> CreateZip[创建 ZIP 文件<br/>project-v1.0.0.zip]
    
    CreateZip --> Upload[上传到 OSS<br/>可选]
    Upload --> Response[返回下载链接]
    
    Response --> End([客户端下载])
    
    style Start fill:#2196F3,color:#fff
    style End fill:#4CAF50,color:#fff
```

## 7. 错误处理流程

### 7.1 翻译失败处理

```mermaid
flowchart TD
    Start[调用 AI API] --> Try{API 调用}
    
    Try -->|成功| SaveTranslation[保存翻译]
    Try -->|失败| CheckError{检查错误类型}
    
    CheckError -->|速率限制| Retry[等待后重试<br/>最多 3 次]
    CheckError -->|网络错误| Retry
    CheckError -->|API 错误| LogError[记录错误日志]
    
    Retry --> RetryCount{重试次数?}
    RetryCount -->|< 3| Try
    RetryCount -->|>= 3| MarkFailed[标记任务失败]
    
    LogError --> MarkFailed
    MarkFailed --> UpdateJob[更新 job status=failed<br/>error_message]
    
    UpdateJob --> NotifyUser[通知用户<br/>WebSocket/邮件]
    
    SaveTranslation --> CheckComplete{所有词条完成?}
    CheckComplete -->|是| Success[任务完成]
    CheckComplete -->|否| Continue[继续下一批]
    
    Continue --> Start
    
    style Success fill:#4CAF50,color:#fff
    style MarkFailed fill:#f44336,color:#fff
```

## 8. 缓存策略

```mermaid
graph TD
    subgraph "读取流程"
        A[请求数据] --> B{Redis 缓存?}
        B -->|命中| C[返回缓存数据]
        B -->|未命中| D[查询数据库]
        D --> E[写入缓存<br/>TTL: 1h]
        E --> F[返回数据]
    end
    
    subgraph "更新流程"
        G[更新数据] --> H[写入数据库]
        H --> I[删除相关缓存]
        I --> J[后续请求重新缓存]
    end
    
    style C fill:#4CAF50,color:#fff
    style E fill:#FF9800
```

### 缓存键设计

```
# 项目信息
project:{projectId}

# 项目的词条列表
project:{projectId}:entries:page:{page}:status:{status}

# 单个词条详情
entry:{entryId}

# 词条的翻译
entry:{entryId}:translation:{locale}

# 项目的所有翻译（用于导出）
project:{projectId}:export:{locale}:version:{versionName}

# 版本列表
project:{projectId}:versions

# 主版本信息
project:{projectId}:master-version
```

## 9. WebSocket 实时更新

```mermaid
sequenceDiagram
    participant Web as 前端
    participant WS as WebSocket Server
    participant Worker as Queue Worker
    participant Redis as Redis PubSub
    
    Web->>WS: 建立 WebSocket 连接
    WS->>WS: 保存连接 (projectId, userId)
    
    Worker->>Worker: 翻译任务进度更新
    Worker->>Redis: PUBLISH translation:progress<br/>{projectId, jobId, progress}
    
    Redis->>WS: 订阅消息
    WS->>WS: 查找订阅该项目的连接
    WS->>Web: emit('translation:progress', data)
    
    Web->>Web: 更新进度条
    
    Note over Worker,WS: 类似的事件：<br/>- entry:created<br/>- entry:updated<br/>- translation:completed<br/>- version:created
```

## 10. 部署架构

```mermaid
graph TB
    subgraph "用户层"
        U1[开发者 CLI]
        U2[浏览器用户]
    end
    
    subgraph "负载均衡层"
        LB[Nginx<br/>负载均衡器]
    end
    
    subgraph "应用层"
        Web1[Web 实例 1]
        Web2[Web 实例 2]
        API1[API 实例 1]
        API2[API 实例 2]
        Worker1[Worker 实例 1]
        Worker2[Worker 实例 2]
    end
    
    subgraph "数据层"
        DB[(MySQL<br/>主从复制)]
        Redis[(Redis<br/>哨兵模式)]
    end
    
    subgraph "外部服务"
        AI[OpenAI API]
        OSS[对象存储<br/>AWS S3]
    end
    
    U1 --> LB
    U2 --> LB
    
    LB --> Web1
    LB --> Web2
    LB --> API1
    LB --> API2
    
    Web1 --> API1
    Web2 --> API2
    
    API1 --> DB
    API2 --> DB
    API1 --> Redis
    API2 --> Redis
    
    Redis --> Worker1
    Redis --> Worker2
    
    Worker1 --> DB
    Worker2 --> DB
    Worker1 --> AI
    Worker2 --> AI
    
    API1 --> OSS
    API2 --> OSS
    
    style LB fill:#4CAF50,color:#fff
    style DB fill:#2196F3,color:#fff
    style Redis fill:#f44336,color:#fff
    style AI fill:#FF9800
```

---

以上流程图涵盖了：
1. **系统架构流程** - 整体组件交互
2. **CLI 工作流程** - 开发者使用 CLI 的完整流程
3. **平台翻译流程** - 手动和 AI 翻译的详细步骤
4. **版本管理流程** - 版本创建、设置和对比
5. **数据流转图** - 数据在系统中的流动
6. **API 调用流程** - 认证和导出流程
7. **错误处理流程** - 异常情况的处理
8. **缓存策略** - 提升性能的缓存设计
9. **WebSocket 实时更新** - 实时通知机制
10. **部署架构** - 生产环境部署方案

您可以使用支持 Mermaid 的工具查看这些流程图，如：
- [Mermaid Live Editor](https://mermaid.live)
- VS Code 插件: Markdown Preview Mermaid Support
- Notion、Confluence 等支持 Mermaid 的文档工具
