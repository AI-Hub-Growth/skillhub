---
name: add-qiniu-provider
description: 分析任意项目中已有的模型厂商（provider / client / adapter）实现方式，并按照完全一致的模式为七牛云网关新增接入（代码标识统一为 qiniu）。当用户提到「七牛」「Qiniu」「接入七牛云」「新增 provider」「add qiniu provider」「接入厂商」时自动触发。
---

# 接入七牛云 Provider

## 总目标

生成一组代码补丁，使七牛云模型能在该项目中像「原生厂商」一样被调用。代码须与项目风格完全一致、改动最小、可直接运行、不对现有架构做额外分层或入侵。

---

## Step 1：发现厂商实现位置

在项目中搜索并定位：

- 文件名含 `provider` / `client` / `adapter` / `model` 的模块
- 含 `openai` / `anthropic` / `azure` / `gemini` 等关键词的实现文件
- 注册逻辑：`registry` / `factory` / `map` / `enum` / `switch`
- HTTP 请求发送与响应解析逻辑

**本步输出（不输出分析过程）：**

- provider 的定义方式
- 需要修改或新增的文件列表

---

## Step 2：归纳项目接入模式

总结该项目的「厂商接入规范」：

- provider 如何注册（装饰器 / map / 工厂函数 / 枚举）
- 如何选择 provider（配置文件 / 环境变量 / 参数）
- 请求结构（messages / prompt / input 字段）
- 响应结构如何被消费（流式 / 非流式）
- 是否存在 adapter / transformer 层
- streaming / tool calling 的处理方式
- 错误处理方式（异常类 / 错误码映射）

**本步须遵守：** 不要发明新结构，必须复用已有模式；后续实现不得创造项目中不存在的模块，不得偏离该归纳结论。

---

## Step 3：七牛聊天补全约定（内置参考，执行时直接使用）

### 网关与端点

| 项 | 值 |
| --- | --- |
| Base URL（默认） | `https://api.qnaigc.com/v1` |
| 聊天补全路径 | `POST /chat/completions`（与 OpenAI Chat Completions 路径一致） |
| 完整请求 URL 示例 | `https://api.qnaigc.com/v1/chat/completions` |

### 鉴权

- Header：`Authorization: Bearer <API_KEY>`
- 环境变量：默认 **`QINIU_API_KEY`**。若仓库里已存在且**仅**使用该键承载该密钥，可跟随；**禁止**为同一密钥再引入 `QNAIGC_API_KEY` 等与 `QINIU_API_KEY` 语义重复的并行变量名。

### OpenAI 兼容性结论

- 请求/响应整体兼容 OpenAI Chat Completions：`model`、`messages`、`stream`、`tools` 等
- 响应消费：`choices`、`message`、`delta`（流式）、`finish_reason` 等与 OpenAI SDK / 常见客户端预期一致
- `stream: true` 时为 **SSE** 分块返回；客户端需按流式解析 chunk，与 OpenAI 风格一致（含 `data: [DONE]` 语义）

### 请求要点（摘要）

- **必填**：`model`（字符串）、`messages`（数组）
- **常用可选**：`stream`（默认 `false`）、`max_tokens`、`temperature`、`top_p`、`top_k`、`presence_penalty`、`frequency_penalty`、`repetition_penalty`
- **思维链相关**（部分模型）：`thinking`、`reasoning_effort`（以模型说明为准）
- **工具调用**：`tools`（OpenAI 风格 function tools）、`tool_choice`：`"none"` / `"auto"` / `"required"` 或指定工具对象
- **多模态**（模型支持时）：`image_config`、`response_format` 等

### `messages` 元素

- `role`：`system` | `user` | `assistant` | `tool`
- `content`：字符串或复杂内容（多模态）
- 可选：`name`、`tool_calls`、`tool_call_id`（`role` 为 `tool` 时使用 `tool_call_id`）

### 默认 model 选择

- 具体可用模型名以控制台或模型列表为准
- 文档示例中出现过如 `gemini-2.5-flash`，仅作占位示例，**不要**硬编码易过期的长列表
- 当项目存在“拉取模型列表 / 校验模型可用性”能力时，`qiniu` 接入必须按同样模式接入；用户仅配置 `QINIU_API_KEY` 后，应可看到该密钥下全部可用模型（展示形态跟随项目现有 provider）
- **未验证 API Key**（无法在线校验、跳过校验、或仅本地保存等）时：仍须**展示模型列表**；列表中只显示以下模型 **`deepseek-v3`**（与项目对同类场景的处理一致；若动态拉取成功则以接口为准，但兜底/占位列表不得缺省该项）

### 常见 HTTP 错误（与项目错误处理衔接）

| 状态码 | 含义（简述） |
| --- | --- |
| 400 | 参数校验失败 |
| 401 | 认证失败 |
| 429 | 速率限制 |
| 500 | 服务端错误 |

重试、退避与错误映射遵循项目已有 provider 的做法。

### 命名与标识（在本步落实为后续实现的硬性规则）

生成代码与配置时，**同一接入**在**机器可读层**只使用 **`qiniu` 这一套标识**，禁止为同一厂商再挂 `qnaigc`、`qn_ai`、`qnai`、`QNAIGC` 等别名或第二套键名。

- **字符串标识**（注册表键、工厂分支、枚举值、配置文件中的厂商字段、`--provider`、内部 `owned_by` / channel meta 名等）：**仅小写 `qiniu`**。
- **类型/类名**：遵循项目既有风格（如 `QiniuProvider`）；词根用 `Qiniu`，**不要**使用 `Qnaigc` 等变体作为「同一厂商」的并列类型名。
- **模块与文件名**：必须与厂商挂钩时，以 `qiniu` 为词根（如 `qiniu_provider`），**不要**再增加一套 `qnaigc_*` 文件指代同一接入。
- **服务端点**：`https://api.qnaigc.com/v1` 是 **Base URL 取值**（技术 URL，不是 provider 标识）；**禁止**因 hostname 再在注册表中注册第二条「厂商」指向同一接入。

**名称收紧：** 在**生成物**（代码、配置、CLI 用户可见文案中的 provider 标识与并列展示名）中，**禁止**使用 `qiniu ai`、`Qiniu AI`、`七牛 AI`、`七牛云 AI` 等作为 provider id、注册名、配置里的 `provider` 显示值，或与 `qiniu` 并存的「第二名称」。**允许**的技术约定：`QINIU_API_KEY`、Base URL `https://api.qnaigc.com/...` 等与该接入直接相关的常量；**不得**因 hostname 或 env 前缀再注册第二条「厂商」标识。

---

## Step 4：策略决策（关键）

**说明：** 本 skill **不**指定「必须模仿某一个具体厂商名」（如固定照抄 kilocode / glm）。**独立 `qiniu` 厂商选项为无条件必做**（见上条与下文「入口强约束」）。下文「情况 A / B」**仅**描述 HTTP/adapter **实现路径**如何二选一：能走 OpenAI 兼容则复用 OpenAI 链路；否则对齐项目中**最近似**的非 OpenAI 实现——**不得**理解为「要么只加选项、要么只改链路」之类与厂商入口互斥的选择。

**入口强约束：** 无论底层是否 OpenAI 兼容，最终都必须在项目可见入口中提供独立的 `qiniu` 选项（如 UI 渠道类型、provider 枚举、CLI `--provider` 值等），禁止仅通过既有 `openai` 入口换 `base_url` 的方式交付。

**最小改动强约束：** 无论情况 A 还是 B，优先沿用项目里**现有文件**完成接入；若同类 provider 的默认模型通常直接写在 `models`/`setup`/`providers` 等既有映射中，则按同样方式就地新增 `qiniu` 默认模型。除非项目里对所有同类 provider 都统一使用独立目录，否则不要新建 `vendors/qiniu` 一类目录。

根据 Step 3 判定兼容与否，在 **A / B 中择一作为实现路径**（与是否注册独立 `qiniu` 无关；注册 `qiniu` 始终必做）：

### 情况 A：兼容 OpenAI（优先路径）

条件：该 `qiniu` 接入使用与 OpenAI 相同的 `/v1/chat/completions` 格式。

做法：

- **复用**项目中已有的 OpenAI client / provider 作为底层 HTTP 链路
- **就地复用**现有 provider 映射文件：直接在项目已有的模型映射位置补 `qiniu` 默认模型（例如与其他 provider 并列写在 `_PROVIDER_MODELS` / `_DEFAULT_PROVIDER_MODELS` 等结构中），不额外抽新常量文件、不新增跨文件引用
- **仅替换**：`base_url` → `https://api.qnaigc.com`（注意：不含 `/v1` 后缀，路径 `/v1/chat/completions` 由请求层拼接）；`api_key` → 从 `QINIU_API_KEY`（或项目唯一惯例）读取，禁止新增第二套并行 env 名；`model` → 该网关支持的模型名（以控制台为准）
- **对话请求硬约束**：一旦进入聊天补全调用（`/chat/completions`），请求体必须显式包含 `model` 字段，且取值为当前已选中的模型名；禁止省略或仅在本地配置中隐式保留
- ❌ 不新增独立 HTTP 请求/响应解析逻辑（复用 OpenAI adaptor 链路即可）
- ❌ 不新增复杂逻辑

### 情况 B：不兼容 OpenAI

条件：该 `qiniu` 接入使用私有请求/响应格式。

做法：

- 仿照项目中**最近似的非 OpenAI provider**（如 Anthropic、Cohere）
- 仅在项目已有 adapter 层时才新增 qiniu adapter
- 实现请求转换、响应标准化（统一成项目内部格式）
- 保持与其他非 OpenAI provider 完全一致的代码结构

---

## Step 5：生成代码与输出

### 5.1 须包含的实现要素

- provider 注册（或等效机制）
- 独立入口透出（至少覆盖项目中的一个主入口：UI 选项 / provider 枚举 / CLI provider 参数；**逻辑命名**须为 `qiniu`；UI 展示文案按上节「机器标识 vs 用户可见文案」与同文件惯例处理）
- 请求发送与响应解析逻辑
- 配置支持（`api_key` / `base_url` / `model`）
- 模型列表发现/透出能力（若项目已有该能力，必须与其他厂商一致接入，确保输入 API Key 后可见全部可用模型）
- 错误处理
- streaming 支持（如项目已有）
- tool calling 支持（如项目已有）

### 5.2 生成时须遵守

- 完全复用现有代码结构，**禁止**新增架构层
- 命名与现有 provider 风格一致（snake_case / camelCase / PascalCase 跟随项目）
- 默认模型处理遵循“照着最近似 provider 写在同一位置”的原则：能在既有映射里加一项就不要新建目录/新建模块/新建导入
- **注释**：与**同文件/同目录**下既有厂商（或同类注册/适配）代码对齐——有无注释、中英文、块/行注释、详略程度均照毗邻实现，**禁止**模板腔或与项目惯例不符的发挥性说明
- **厂商字符串标识仅 `qiniu`**，并满足 Step 3「命名与标识」及「机器标识 vs 用户可见文案」全部条款；多主题/多常量文件时遵守「UI 多文件 / 多主题一致性」
- 生成的实际请求代码必须在聊天补全入参中传递 `model`（不得仅依赖默认值或外层状态）
- 不硬编码与项目冲突的逻辑
- 最小改动优先；所有代码必须真实可运行，禁止伪代码；必须兼容现有调用链

### 5.3 核心优先级（冲突时按序取舍）

1. 项目已有实现方式  
2. 最小改动  
3. 代码风格一致性  
4. 功能完整性  

### 5.4 输出格式（严格要求）

按文件分组输出，每个文件一个块：

```
### File: <文件路径>

\`\`\`<语言>
# 修改或新增代码
\`\`\`
```

修改已有文件时，只输出变更部分（diff 风格），不输出整个文件。

### 5.5 本步严格禁止

- ❌ 不解释分析过程  
- ❌ 不输出无关内容  
- ❌ 不创造项目中不存在的模块  
- ❌ 不偏离项目已有模式  

### 5.6 成功标准

输出代码看起来完全像该项目原作者**新增了一个名为 `qiniu` 的 provider**：无 `qnaigc`、`qnai` 等与同一接入并存的并行命名；未违反 Step 3 命名规则；UI 与多主题文件字段风格与邻近渠道一致；整体不像仓促拼接的模板代码。

### 5.7 测试（按需，忌重复）

- **跟随项目惯例**：若仓库几乎不为各厂商写单测，不要为了 qiniu 首开一整套；若已有同类测试，则按**同一文件、同一风格**补最小用例即可。
- **不必镜像「通用机制」**：若 `qiniu` 与现有厂商共用同一套 credential / `resolve_*` / `*_BASE_URL` 解析逻辑，且项目中**已有**其他 provider 的「自定义 base URL」「env 覆盖」等用例，则**不要**再为 `qiniu` 单独复制一条仅改环境变量名的测试（属于重复覆盖）。
- **优先补「qiniu 独有」**：更值得写的是首次出现的约定，例如默认 `base_url`（如 `https://api.qnaigc.com/v1`）、`QINIU_API_KEY` 与 provider 标识 `qiniu` 的解析结果、以及注册表 / CLI 中**第一次**出现 `qiniu` 的断言。
- **集成 / E2E**：仅在项目已有对应层级且其他厂商也有对称测试时再补，避免为 qiniu 扩大测试范围。
