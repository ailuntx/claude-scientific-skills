---
name: bgpt-paper-search
description: 通过BGPT MCP服务器搜索科学论文并获取从全文研究中提取的结构化实验数据。每篇论文返回25+个字段，包括方法、结果、样本量、质量评分和结论。适用于文献综述、证据整合及查找摘要中未提供的实验细节。
allowed-tools: Bash
license: MIT
metadata:
    skill-author: BGPT
    website: https://bgpt.pro/mcp
    github: https://github.com/connerlambden/bgpt-mcp
---

# BGPT 论文搜索

## 概述

BGPT是一个远程MCP服务器，其搜索的精选科学论文数据库基于从全文研究中提取的原始实验数据构建。与传统文献数据库仅返回标题和摘要不同，BGPT返回论文实际内容的结构化数据——包括方法、量化结果、样本量、质量评估及每篇论文25+个元数据字段。

## 使用场景

在以下情况使用本技能：
- 搜索包含特定实验细节的科学论文
- 进行系统性或范围性文献综述
- 查找跨研究的量化结果、样本量或效应量
- 比较不同研究采用的方法论
- 寻找带有质量评分或证据分级的论文
- 需要全文论文的结构化数据（不仅限于摘要）
- 为荟萃分析或临床指南构建证据表

## 配置

BGPT是远程MCP服务器——无需本地安装。

### Claude Desktop / Claude Code

在MCP配置中添加：

```json
{
  "mcpServers": {
    "bgpt": {
      "command": "npx",
      "args": ["mcp-remote", "https://bgpt.pro/mcp/sse"]
    }
  }
}
```

### npm (替代方案)

```bash
npx bgpt-mcp
```

## 使用方法

配置完成后，使用BGPT MCP服务器提供的`search_papers`工具：

```
搜索主题为："人类细胞中CRISPR基因编辑效率"的论文
```

服务器返回结构化结果，包括：
- **标题、作者、期刊、年份、DOI**
- **方法**：实验技术、模型、方案
- **结果**：含量化数据的关键发现
- **样本量**：受试者/样本数量
- **质量评分**：研究质量评估
- **结论**：作者结论及意义

## 定价

- **免费版**：每个网络50次搜索，无需API密钥
- **付费版**：每条结果$0.01，需从[bgpt.pro/mcp](https://bgpt.pro/mcp)获取API密钥
