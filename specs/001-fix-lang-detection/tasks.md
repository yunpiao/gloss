# Tasks: 修复中文页面注解目标词汇问题

**Branch**: `001-fix-lang-detection`
**Created**: 2025-12-01
**Spec**: [spec.md](./spec.md) | **Plan**: [plan.md](./plan.md)

## Task Summary

| ID | Task | Status | Priority | Est. |
|----|------|--------|----------|------|
| T1 | 修改中文页面 prompt | ✅ Done | P1 | 10min |
| T2 | 手动测试验证 | 🔄 In Progress | P1 | 15min |

---

## Phase 1: Implementation

### T1: 修改 fetchDictionary 函数中的中文页面 prompt

**Priority**: P1 (Core Fix)
**Status**: ✅ Done
**File**: `a.js`
**Lines**: 400-424

**Description**:
修改中文页面的 prompt，添加 CRITICAL RULES 部分，明确指示 AI 只提取中文词汇（汉字），排除页面中已有的英文单词。

**Changes**:

将当前 prompt (lines 400-418):
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

替换为:
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

IMPORTANT: Return ONLY raw JSON. Do NOT wrap in markdown code blocks. Do NOT add any explanation or text before/after the JSON.

Output format:
{"中文词1": "english1", "中文词2": "english2", ...}

Text to analyze:
"""
${text}
"""`
```

**Acceptance Criteria**:
- [x] Prompt 包含 CRITICAL RULES 部分
- [x] 明确排除英文单词、数字、品牌名
- [x] 保持输出格式不变
- [x] 不影响英文页面的 prompt

---

## Phase 2: Verification

### T2: 手动测试验证

**Priority**: P1
**Status**: ⬜ Pending
**Dependencies**: T1

**Test Scenarios**:

1. **中文页面测试**
   - [ ] 访问知乎/百度等中文网站
   - [ ] 触发注解功能
   - [ ] 验证所有被注解词汇均为中文
   - [ ] 验证页面中的英文单词未被注解

2. **中英混合页面测试**
   - [ ] 访问包含英文品牌名/技术术语的中文页面
   - [ ] 触发注解功能
   - [ ] 验证英文内容被正确忽略

3. **英文页面回归测试**
   - [ ] 访问英文网站（如 Wikipedia）
   - [ ] 触发注解功能
   - [ ] 验证功能与修复前一致

**Success Criteria**:
- [ ] SC-001: 中文页面 100% 被注解词汇为中文
- [ ] SC-002: 中文页面注解内容为英文翻译
- [ ] SC-003: 英文页面功能不受影响

---

## Completion Checklist

- [ ] T1: 代码修改完成
- [ ] T2: 所有测试场景通过
- [ ] 代码已提交到 `001-fix-lang-detection` 分支
