---
name: serper-scrape
description: 调用 Serper API 搜索和抓取网页内容。支持两种模式：(1) 搜索模式 - 根据关键词搜索网页，支持时间范围筛选；(2) 抓取模式 - 获取指定 URL 的 markdown 内容。当用户说"搜索"、"查找"、"search"、"抓取网页"、"获取网页内容"、"scrape"时使用此 skill。
---

# Serper Search & Scrape Skill

使用 Serper API 进行网页搜索和内容抓取。

## 环境变量

使用前请设置环境变量：
```bash
export SERPER_API_KEY="your-api-key"
```

或在 shell 配置文件中添加：
```bash
# ~/.zshrc 或 ~/.bashrc
export SERPER_API_KEY="a41b4402bea6ca7647f39817ae028fce6c86e7f6"
```

---

## 功能一：搜索 (Search)

根据关键词搜索网页，返回搜索结果列表。

### API 配置

- **Endpoint**: `https://google.serper.dev/search`
- **Method**: POST
- **Headers**:
  - `X-API-KEY`: 从环境变量 `SERPER_API_KEY` 读取
  - `Content-Type`: `application/json`

### 请求参数

| 参数 | 类型 | 必需 | 说明 |
|------|------|------|------|
| `q` | string | 是 | 搜索关键词 |
| `gl` | string | 否 | 国家代码，如 `us`、`cn`、`jp`，默认 `us` |
| `hl` | string | 否 | 语言代码，如 `en`、`zh-cn`、`ja`，默认 `en` |
| `tbs` | string | 否 | 时间范围筛选（见下表） |
| `num` | number | 否 | 返回结果数量，默认 10，最大 100 |
| `page` | number | 否 | 页码，默认 1 |

### 时间范围 (tbs) 参数值

| 值 | 含义 |
|----|------|
| `qdr:h` | 过去 1 小时 |
| `qdr:d` | 过去 24 小时 |
| `qdr:w` | 过去 1 周 |
| `qdr:m` | 过去 1 个月 |
| `qdr:y` | 过去 1 年 |

### 使用示例

```bash
# 基础搜索
curl --location 'https://google.serper.dev/search' \
--header "X-API-KEY: $SERPER_API_KEY" \
--header 'Content-Type: application/json' \
--data '{"q":"apple inc"}'

# 搜索过去一个月的内容
curl --location 'https://google.serper.dev/search' \
--header "X-API-KEY: $SERPER_API_KEY" \
--header 'Content-Type: application/json' \
--data '{"q":"OpenAI GPT-5","tbs":"qdr:m","gl":"us","hl":"en"}'

# 中文搜索
curl --location 'https://google.serper.dev/search' \
--header "X-API-KEY: $SERPER_API_KEY" \
--header 'Content-Type: application/json' \
--data '{"q":"人工智能最新进展","gl":"cn","hl":"zh-cn","tbs":"qdr:w"}'
```

### 响应结构

```json
{
  "searchParameters": {
    "q": "apple inc",
    "gl": "us",
    "hl": "en",
    "type": "search"
  },
  "knowledgeGraph": {
    "title": "Apple",
    "type": "Technology company",
    "description": "...",
    "attributes": { ... }
  },
  "organic": [
    {
      "title": "Apple",
      "link": "https://www.apple.com/",
      "snippet": "Discover the innovative world of Apple...",
      "position": 1
    },
    ...
  ],
  "peopleAlsoAsk": [ ... ],
  "relatedSearches": [ ... ]
}
```

---

## 功能二：抓取 (Scrape)

获取指定 URL 的完整页面内容，返回 markdown 格式。

### API 配置

- **Endpoint**: `https://scrape.serper.dev`
- **Method**: POST
- **Headers**:
  - `X-API-KEY`: 从环境变量 `SERPER_API_KEY` 读取
  - `Content-Type`: `application/json`

### 请求参数

| 参数 | 类型 | 必需 | 说明 |
|------|------|------|------|
| `url` | string | 是 | 目标网页完整 URL |
| `includeMarkdown` | boolean | 否 | 返回 markdown 格式，默认 `true` |

### 使用示例

```bash
curl --location 'https://scrape.serper.dev' \
--header "X-API-KEY: $SERPER_API_KEY" \
--header 'Content-Type: application/json' \
--data '{"url":"https://apple.com","includeMarkdown":true}'
```

### 响应结构

```json
{
  "markdown": "页面的 markdown 内容...",
  "text": "页面的纯文本内容...",
  "metadata": { ... }
}
```

---

## 组合工作流：搜索 + 抓取

**推荐流程**：先搜索获取相关链接，再抓取感兴趣的页面内容。

### 步骤

1. **搜索**：使用 search API 根据关键词获取结果列表
2. **筛选**：从 `organic` 结果中选择最相关的链接
3. **抓取**：对选中的链接调用 scrape API 获取完整内容

### 示例场景

用户请求："搜索最近一周关于 Claude 4 的新闻，并获取详细内容"

**执行步骤**：

```bash
# 步骤 1: 搜索
curl --location 'https://google.serper.dev/search' \
--header "X-API-KEY: $SERPER_API_KEY" \
--header 'Content-Type: application/json' \
--data '{"q":"Claude 4 Anthropic","tbs":"qdr:w","num":5}'

# 步骤 2: 从搜索结果中提取 organic[].link

# 步骤 3: 抓取每个感兴趣的链接
curl --location 'https://scrape.serper.dev' \
--header "X-API-KEY: $SERPER_API_KEY" \
--header 'Content-Type: application/json' \
--data '{"url":"<从搜索结果获取的链接>","includeMarkdown":true}'
```

---

## 典型用例

1. **时效性搜索**：搜索最近的新闻、公告、更新
2. **深度研究**：搜索主题 → 抓取多个来源 → 综合分析
3. **竞品分析**：搜索竞品信息 → 抓取详情页
4. **内容获取**：直接抓取已知 URL 的内容
5. **多语言搜索**：通过 `gl` 和 `hl` 参数搜索不同语言/地区的内容

---

## 限制规则（必须遵守）

### 1. 抓取数量限制

**每次搜索最多抓取 5 个网页**，以降低 API 调用成本。

- 如果搜索结果超过 5 条，只选择最相关的前 5 个进行抓取
- 如用户明确要求更多，需告知限制原因并确认是否继续

### 2. 内容保存规则

**所有抓取的网页内容必须处理后保存到 Obsidian**。

#### 保存目录分类

| 内容类型 | 保存目录 |
|---------|---------|
| 普通内容（新闻、观点、案例等） | `/Users/liuxingqi/tars/Resources/Web-Search/` |
| 技术/工具类（官网文档、API、Wiki 等） | `/Users/liuxingqi/tars/Knowledge/` |

#### 判断技术类内容的标准

以下类型归类为技术/工具类，保存到 `knowledge` 目录：
- 官方文档、API 文档
- GitHub README、Wiki
- 技术教程、配置指南
- 工具使用说明
- 编程语言参考手册

### 3. 内容处理规则

抓取的原始内容必须经过处理，**保留有价值信息，去除噪音**。

#### 必须保留

- ✅ 完整的案例和示例
- ✅ 核心观点和论点
- ✅ 事实和数据
- ✅ 金句和精华语录
- ✅ 引用的资料链接
- ✅ 原文网站链接（作为来源标注）

#### 必须去除

- ❌ 导航菜单、页脚
- ❌ 广告内容
- ❌ 图片链接（除非是关键图表）
- ❌ 视频嵌入链接
- ❌ 社交分享按钮
- ❌ 评论区（除非评论包含有价值信息）
- ❌ 相关推荐列表
- ❌ Cookie 提示、弹窗内容

### 4. 保存文件格式

```markdown
---
title: {{页面标题}}
source: {{原始URL}}
date_scraped: {{抓取日期 YYYY-MM-DD}}
tags: [web-search, {{主题标签}}]
---

# {{页面标题}}

> 来源：[{{网站名称}}]({{原始URL}})

## 一句话总结

{{核心要点概括}}

## 关键内容

{{处理后的正文内容}}

## 金句/要点

- {{提取的金句或关键观点}}

## 引用链接

- [{{链接标题}}]({{URL}})
```

---

## 注意事项

- 确保环境变量 `SERPER_API_KEY` 已设置
- 搜索时合理使用 `tbs` 参数缩小时间范围，提高结果相关性
- 抓取 URL 必须是完整路径，包含 `https://` 前缀
- 某些网站可能有反爬措施，抓取失败时告知用户
- 搜索+抓取组合会消耗多次 API 调用，注意配额
- **每次最多抓取 5 个网页**
- **所有内容必须处理后保存到对应 Obsidian 目录**
