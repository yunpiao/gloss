# Implementation Plan: 修复中文页面注解目标词汇问题

**Branch**: `001-fix-lang-detection` | **Date**: 2025-12-01 | **Spec**: [spec.md](./spec.md)
**Input**: Feature specification from `/specs/001-fix-lang-detection/spec.md`

## Summary

修复中文页面上注解目标词汇错误的问题。当前系统在中文页面上错误地为页面中的英文单词添加注解，而正确的行为应该是为中文词汇添加英文翻译注解。

**核心问题**: `fetchDictionary` 函数中的中文页面 prompt 要求 AI 提取中文词汇并返回英文翻译，但返回的字典格式 `{"中文词": "english"}` 在 `applyDictionaryToPage` 中被正确处理。问题在于 prompt 的措辞可能导致 AI 错误地提取了页面中的英文单词而非中文词汇。

**修复方案**: 修改中文页面的 prompt，明确指示 AI 只提取中文词汇（排除英文单词），并返回对应的英文翻译。

## Technical Context

**Language/Version**: JavaScript (ES6+, Tampermonkey UserScript)
**Primary Dependencies**: Tampermonkey API (GM_xmlhttpRequest, GM_setValue, GM_getValue, GM_addStyle, GM_registerMenuCommand)
**Storage**: Tampermonkey GM_setValue/GM_getValue (浏览器本地存储)
**Testing**: 手动测试（浏览器扩展环境，无自动化测试框架）
**Target Platform**: 浏览器 (Chrome/Firefox/Edge with Tampermonkey)
**Project Type**: Single file userscript
**Performance Goals**: 保持现有性能，API 响应时间取决于外部服务
**Constraints**: 单文件脚本，无构建流程，需兼容 Tampermonkey 沙箱环境
**Scale/Scope**: 单用户本地使用

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

项目 constitution 为模板状态（未配置具体原则），因此无特定约束需要检查。本修复遵循以下通用原则：

- [x] 最小化修改：只修改必要的代码（prompt 文本）
- [x] 保持向后兼容：不改变现有配置和用户数据
- [x] 不引入新依赖：使用现有技术栈

## Project Structure

### Documentation (this feature)

```text
specs/001-fix-lang-detection/
├── spec.md              # Feature specification
├── plan.md              # This file
├── research.md          # Phase 0 output
└── tasks.md             # Phase 2 output (created by /speckit.tasks)
```

### Source Code (repository root)

```text
a.js                     # Main userscript file (single file project)
```

**Structure Decision**: 单文件 Tampermonkey 脚本，所有代码在 `a.js` 中。无需创建额外目录结构。

## Bug Analysis

### 问题定位

**文件**: `a.js`
**函数**: `fetchDictionary` (lines 394-489)
**具体位置**: lines 400-418 (中文页面 prompt)

### 当前代码行为

```javascript
const prompt = isChinese
    ? `You are a language learning assistant. Analyze the following Chinese text and identify ${config.wordCount} difficult or important Chinese words or short phrases that a learner might not know.

Requirements:
1. Return a JSON object where keys are Chinese words or short phrases (exactly as they appear in the text)
2. Values are concise English translations (1-4 words maximum)
...`
```

**问题**: prompt 虽然说明要提取 "Chinese words"，但没有明确排除页面中已有的英文单词。AI 可能会将页面中的英文单词也作为"需要学习的词汇"返回。

### 修复方案

修改中文页面的 prompt，添加明确指令：
1. 只提取**中文词汇**（汉字组成的词）
2. 明确排除页面中已有的英文单词、品牌名、技术术语等
3. 返回格式保持不变：`{"中文词汇": "英文翻译"}`

## Implementation Steps

### Step 1: 修改 fetchDictionary 函数中的中文 prompt

**位置**: `a.js` lines 400-418

**修改内容**:
- 在 Requirements 中添加明确指令：只提取中文词汇（由汉字组成）
- 添加排除规则：忽略页面中的英文单词、数字、标点
- 强调 keys 必须是纯中文字符

### Step 2: 验证修复

**测试场景**:
1. 访问中文网页（如知乎、百度）
2. 触发注解功能
3. 验证被注解的词汇全部为中文词汇
4. 验证页面中的英文单词未被注解

## Complexity Tracking

> 无 Constitution 违规需要记录

| Violation | Why Needed | Simpler Alternative Rejected Because |
|-----------|------------|-------------------------------------|
| N/A | N/A | N/A |

## Risk Assessment

| Risk | Likelihood | Impact | Mitigation |
|------|------------|--------|------------|
| AI 仍然提取英文单词 | Low | Medium | 在 prompt 中使用更强的排除语言 |
| 修改影响英文页面功能 | Very Low | High | 只修改 isChinese 分支的 prompt |
| 中文词汇提取数量减少 | Low | Low | 可接受，质量优先于数量 |
