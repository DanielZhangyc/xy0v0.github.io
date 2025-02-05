---
date: 2025-02-05T14:25:28+08:00
tags:
  - TypeScript
  - Obsidian
title: 在obsidian中实现文章概括生成
slug: "1738736728"
share: true
searchHidden: false
disableShare: true
canonicalURL: ""
keywords: 
description: 文章讨论了在使用 Obsidian 和 Hugo 配置博客时，为文章生成概括内容的实现方法。作者通过在 Obsidian 中使用 requesturl 库向 OpenAI 发送请求，从而获取文章的自动摘要。为适应不同用户的需求，系统允许用户个性化设置添加的字段名，并通过正则表达式对 formatter 内容进行修改。此外，文章详细描述了处理前端内容的流程，包括创建或更新 frontmatter 和 summaryField 字段的步骤。错误处理方面，当API请求遇到问题时，系统会抛出相应的错误类型并提示用户。最终，该功能实现了博客文章摘要的自动生成，提升了阅读和检索的便利性。
series: 系列
lastmod: 
lang: cn
cover:
    image: 
author: xy0v0
dir: posts
---
## 开发缘由

在使用 obsidian+hugo 的博客配置时，发现需要文章的概括内容，于是决定直接从 obsidian 端实现该功能。

## 主要实现

### openai请求

采用 obsidian 推荐的 requesturl 库进行大模型请求


```TS
export class OpenAIService {
    constructor(apiKey: string, baseUrl: string, model: string, systemPrompt: string) {
        this.apiKey = apiKey;
        this.baseUrl = baseUrl;
        this.model = model;
        this.systemPrompt = systemPrompt;
    }

    private async makeRequest(messages: ChatMessage[]): Promise<string> {
        try {
            const response = await requestUrl({
                url: `${this.baseUrl}/v1/chat/completions`,
                method: 'POST',
                headers: {
                    'Authorization': `Bearer ${this.apiKey}`,
                    'Content-Type': 'application/json',
                },
                body: JSON.stringify({
                    model: this.model,
                    messages,
                    temperature: 0.7,
                    max_tokens: COMPLETION_TOKENS,
                }),
            });
            // 处理响应...
        } catch (error) {
            // 错误处理...
        }
    }
}
```

## formatter内容修改处理

考虑到添加的字段名有个性化需求，因此在设置中可让用户自行设定添加的字段名，对多种不同的格式有较好适配性。

针对formatter修改有如下方案可选：
- 使用YAML解析器
- 正则表达式

个人对正则表达式比较熟悉，最终采用正则表达式进行

```mermaid
flowchart TD
    A[开始] --> B{检查frontmatter}
    B -->|不存在| C[创建新的frontmatter]
    B -->|存在| D{检查summaryField字段}
    
    D -->|存在| E[使用已有字段]
    D -->|不存在| F[准备创建新字段]
    
    C --> G[生成AI摘要]
    E --> G
    F --> G
    
    G --> H{字段是否存在}
    H -->|存在| I[替换原有内容]
    H -->|不存在| J[添加新字段]
    
    I --> K[保持格式]
    J --> K
    
    K --> L[更新文档]
    L --> M[结束]
```
```TS
private static readonly FRONTMATTER_REGEX = /^---\n([\s\S]*?)\n---\n*/;
private static readonly FIELD_REGEX = (field: string) =>
new RegExp(`^${field}:([\\s\\S]*?)(?=\\n[^\\s]|$)`, 'm');
```

### 错误处理

请求遇到错误时抛出相对应的错误类型

```TS
   try {
       // API调用
   } catch (error) {
       if (error.status === 429) {
           new Notice('API rate limit exceeded. Please try again later.');
       } else if (error.status === 401) {
           new Notice('Invalid API key. Please check your settings.');
       } else {
           new Notice(`API Error: ${error.message}`);
       }
       throw error;
   }
```

## 总结

经过开发，终于可以为博客文章自动生成描述，方便阅读和后续检索

开源地址：[GitHub](https://github.com/DanielZhangyc/Auto-AI-Summary)