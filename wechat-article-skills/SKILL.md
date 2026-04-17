---
name: wechat-article-skills
description: 微信公众号文章排版工作流：从已有公众号文章中提取视觉样式，将 Markdown 内容排版为微信兼容 HTML，并通过微信公众号 API 推送到草稿箱。当用户提到「公众号排版」「微信排版」「提取公众号样式」「推送到草稿箱」「wechat typesetting」时触发。依赖 wechat-typesetting 项目（https://github.com/inhai-wiki/wechat-typesetting）。
---

# 微信公众号文章排版工作流

## 总目标

提供一套完整的微信公众号排版流程，包含三个可独立使用的子技能：

| 子技能 | 触发词 | 功能 |
|--------|--------|------|
| `extract-wechat-style` | 提取样式、style extract | 从公众号文章 URL 提取视觉 DNA，保存为样式模板 |
| `format-article` | 排版、format、Markdown 转公众号 | 用样式模板将 Markdown 排版为微信兼容 HTML |
| `publish-to-wechat` | 推送草稿、发布公众号、publish draft | 将排版好的 HTML 推送到公众号草稿箱 |

## 前置条件

- 已克隆并启动 [wechat-typesetting](https://github.com/inhai-wiki/wechat-typesetting) 项目
- Node.js 环境可用

启动服务：

```bash
cd <wechat-typesetting-project-root> && npm run dev
# 访问 http://localhost:3000
```

---

## 子技能 1：提取微信公众号文章样式

从微信公众号文章 URL 中提取排版样式，保存为可复用的样式模板。

### 执行步骤

1. **启动服务**（已启动则跳过）
2. **提取样式**：在「样式提取」Tab 中粘贴微信文章链接，点击「提取样式」
3. **验证样式**：检查右侧「样式信息」面板
   - 主色 / 文字色 / 强调色
   - 字体和字号
   - 标题对齐方式和装饰类型
   - CSS 规则
4. **保存模板**：点击「保存为模板」，输入模板名称

### 技术实现

- **抓取**：`src/lib/fetcher.ts` — 服务端 fetch，伪装浏览器 UA
- **解析**：`src/lib/parser.ts` — cheerio 解析 `#js_content` 区域，提取 StyleProfile
  - 配色：`primaryColor`, `textColor`, `accentColors`
  - 字体：`fontFamily`, `bodyFontSize`, `headingFontSize`
  - 装饰：`headingDecoration`（`left-border` / `bottom-border` / `none`）
  - 引用：`quoteStyle`（`borderLeft`）
- **存储**：`src/lib/storage.ts` — JSON 写入 `data/templates/`

### 限制

- 只支持 `mp.weixin.qq.com` 域名的文章链接
- 文章懒加载图片自动转换：`data-src → src`
- 提取的是「视觉 DNA」（配色/字体/装饰模式），不是原始 CSS 选择器

---

## 子技能 2：Markdown 文章排版美化

根据已保存的样式模板，将 Markdown 文章排版为微信公众号兼容格式。

### 执行步骤

1. **启动服务**（已启动则跳过），切换到「Markdown 排版」Tab
2. **选择模板**：右上角下拉框选择已保存的样式模板（或使用默认样式）
3. **输入 Markdown**：在左侧输入框粘贴内容，第一个 `# 标题` 自动提取为文章标题
4. **点击「排版」**：预览区显示效果
5. **导出**：
   - 「一键复制到公众号」— 内联样式 HTML，可直接粘贴到公众号编辑器
   - 「下载 HTML」— 下载为 `.html` 文件

### 微信兼容性规则（核心）

#### 标签转换

- `<ul>/<ol>/<li>` → 转为 `<section>` + 手动编号/符号（微信会重新排版列表）
- `<blockquote>` → 转为 `<section>` + `border-left`（微信会把 blockquote 变色块）
- `<p>` → 外层包 `<section>` 控制间距（微信会给 `p` 加额外 margin）

#### 禁止使用的 CSS

- `calc()` → 预计算为固定 px 值
- `#xxx80` hex+alpha → 改用标准颜色
- `display:flex` → 改用 `<table>` 布局
- `background` 色块 → 引用/表头/标题不要用 background

#### 多图横排

同行多张图片用 `<table>` 包裹：

```markdown
![图1](url1) ![图2](url2)
```

自动转为每张图一个 `<td>` 的表格布局。

### 技术实现

- **核心函数**：`buildInlineHtml()` in `src/components/MarkdownFormatter.tsx`
- **原理**：DOMParser 解析 marked 输出 → 遍历 DOM 树 → 按标签名写入内联 style
- 预览和推送使用**同一套**逻辑，确保所见即所得

---

## 子技能 3：发布到微信公众号草稿箱

将排版好的文章推送到微信公众号草稿箱。

### 前置条件

1. 已完成排版（子技能 2）
2. 已在「设置」中配置公众号 AppID 和 AppSecret
3. 当前公网 IP 已加入微信公众平台的 IP 白名单

### 首次配置

#### 获取 AppID 和 AppSecret

登录 [微信公众平台](https://mp.weixin.qq.com/) →「设置与开发」→「基本配置」

#### 配置 IP 白名单

「基本配置」→「IP 白名单」→ 添加当前公网 IP

查看当前 IP：

```bash
curl -4 -s https://checkip.amazonaws.com
```

#### 保存配置

工具右上角齿轮图标 → 输入 AppID 和 AppSecret → 保存

配置保存在本地 `data/config.json`，不会上传。

### 发布步骤

1. 在「Markdown 排版」Tab 完成排版后，点击「推送到草稿箱」
2. 弹窗确认文章标题（可修改作者）
3. 点击「推送到草稿箱」，等待完成
4. 登录微信公众平台 →「内容管理」→「草稿」中验证

### 技术实现（API 流程）

```
1. GET  /cgi-bin/token?grant_type=client_credential&appid=APPID&secret=APPSECRET
2. POST /cgi-bin/media/uploadimg          # 上传文内图片，替换为微信图床 URL
3. POST /cgi-bin/material/add_material?type=image  # 上传封面图
4. POST /cgi-bin/draft/add               # 创建草稿
   Body: { articles: [{ title, content, thumb_media_id, author }] }
```

代码位置：

- 配置管理：`src/lib/config.ts` + `src/app/api/config/route.ts`
- 微信 API：`src/lib/wechat.ts`（token 管理、图片上传、草稿创建）
- 推送入口：`src/app/api/publish/draft/route.ts`
- 前端弹窗：`src/components/PublishDialog.tsx`

### 常见问题

| 错误 | 原因 | 解决 |
|------|------|------|
| `invalid ip` | IP 未加白名单 | 在公众平台添加当前 IP |
| `access_token 失败` | AppID/AppSecret 错误 | 检查配置 |
| `图片上传失败` | 图片 URL 无法访问 | 确保图片可公网访问 |
| 样式丢失 | HTML 含 `<style>` 标签 | 排版引擎已自动转为内联样式 |

**注意**：

- access_token 有效期 2 小时，有缓存机制自动续期
- 文内图片会自动上传到微信图床，替换原始 URL
- 家庭宽带 IP 可能变动，需重新更新白名单

---

## 版权声明

本 skill 改编自 [wechat-article-skills](https://github.com/inhai-wiki/wechat-article-skills)，原项目采用 MIT License，版权归原作者所有。
