# Research: 修复中文页面注解目标词汇问题

**Date**: 2025-12-01
**Feature**: 001-fix-lang-detection

## 问题根因分析

### 代码审查结果

**文件**: `a.js`
**问题函数**: `fetchDictionary` (lines 394-489)

#### 当前中文页面 prompt (lines 400-418)

```javascript
const prompt = isChinese
    ? `You are a language learning assistant. Analyze the following Chinese text and identify ${config.wordCount} difficult or important Chinese words or short phrases that a learner might not know.

Requirements:
1. Return a JSON object where keys are Chinese words or short phrases (exactly as they appear in the text)
2. Values are concise English translations (1-4 words maximum)
3. Skip very common function words and particles
4. Focus on: academic words, domain-specific terms, idioms, set phrases
5. Prioritize words that appear multiple times or are central to the text meaning

IMPORTANT: Return ONLY raw JSON. Do NOT wrap in markdown code blocks. Do NOT add any explanation or text before/after the JSON.

Output format:
{"中文词1": "english1", "中文词2": "english2", ...}

Text to analyze:
"""
${text}
"""`
```

#### 问题分析

1. **Prompt 措辞不够明确**: 虽然说 "Chinese words"，但没有明确定义什么是中文词汇
2. **缺少排除规则**: 没有明确告诉 AI 排除页面中已有的英文单词
3. **AI 可能的误解**: AI 可能认为页面中的英文单词（如品牌名、技术术语）也是"学习者需要知道的词汇"

### 对比英文页面 prompt (lines 419-436)

英文页面的 prompt 明确说明：
- "keys are English words (lowercase, base form/stem)"
- "Skip very common words (the, is, are, have, etc.)"

这种明确的定义在中文 prompt 中缺失。

## 解决方案

### Decision: 修改中文页面 prompt

**Rationale**:
- 最小化修改，只改变 prompt 文本
- 不影响现有代码逻辑和数据结构
- 直接解决问题根因

**Alternatives considered**:
1. ❌ 在 `applyDictionaryToPage` 中过滤非中文 keys - 增加代码复杂度，且问题应在源头解决
2. ❌ 修改语言检测逻辑 - 语言检测本身没有问题，问题在于 prompt
3. ✅ 修改 prompt 明确指令 - 最简单直接的解决方案

### 修改后的 prompt 设计

```javascript
const prompt = isChinese
    ? `You are a language learning assistant helping Chinese users learn English. Analyze the following Chinese text and identify ${config.wordCount} Chinese words or phrases that would be useful for learning their English equivalents.

CRITICAL RULES:
- ONLY extract Chinese words (words composed entirely of Chinese characters 汉字)
- DO NOT extract any English words, numbers, or punctuation that appear in the text
- DO NOT extract brand names, product names, or technical terms written in English/Latin letters
- Each key in the output MUST be pure Chinese characters only

Requirements:
1. Return a JSON object where keys are Chinese words/phrases (汉字 only, no English/numbers)
2. Values are concise English translations (1-4 words maximum)
3. Skip very common function words and particles (的、是、在、了、etc.)
4. Focus on: academic words, domain-specific terms, idioms, set phrases
5. Prioritize words that appear multiple times or are central to the text meaning

IMPORTANT: Return ONLY raw JSON. Do NOT wrap in markdown code blocks.

Output format:
{"中文词1": "english1", "中文词2": "english2", ...}

Text to analyze:
"""
${text}
"""`
```

### 关键改进点

| 改进 | 说明 |
|------|------|
| 添加 CRITICAL RULES 部分 | 明确排除英文单词、数字、标点 |
| 强调 "汉字 only" | 使用中文术语"汉字"避免歧义 |
| 明确排除品牌名/技术术语 | 防止 AI 提取页面中的英文内容 |
| 保持输出格式不变 | 确保与现有代码兼容 |

## 验证计划

### 测试场景

1. **纯中文页面**: 验证正常提取中文词汇
2. **中英混合页面**: 验证只提取中文词汇，忽略英文
3. **技术博客**: 验证忽略代码片段和技术术语中的英文
4. **英文页面**: 验证不影响现有英文页面功能

### 预期结果

- 中文页面：100% 被注解词汇为中文
- 英文页面：行为不变
