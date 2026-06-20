---
name: shuashuati-quiz
description: >
  在刷刷题(shuashuati.com)题库编辑页面自动答题。读取题目和选项，判断正确答案，
  将翻译和解析填入"参考解析"字段，保存并进入下一题。触发场景：用户在刷刷题页面
  要求答题、填写解析、批量做题。
---

# 刷刷题自动答题 Skill

## 前置条件

必须加载 web-access skill 并遵循其指引，通过 CDP 操作浏览器。

## 识别目标页面

URL 包含 `shuashuati.com/user/paper/editor` 且页面含"参考解析"编辑器。

## 解析填入格式（严格遵守）

所有内容**一次性**插入，用 `\n` 分隔三段：

```
【翻译】<题干中文翻译>
【选项翻译】A. <单词> <中文> ｜ B. <单词> <中文> ｜ C. <单词> <中文> ｜ D. <单词> <中文>
【解析】<语法/词汇解析，说明正确答案及排除理由>
```

**示例：**
```
【翻译】该公司试图用炼乳代替奶粉来发布一种改良的牛奶巧克力棒，但它的表现并不比以前的产品好。
【选项翻译】A. weighed 称重、权衡 ｜ B. fared 表现、进行 ｜ C. cared 关心、在意 ｜ D. posed 提出、摆姿势
【解析】fare 作动词意为表现、进行，fare no better than 表示表现不比……好。weighed 意为称重、权衡；cared 意为关心、在意；posed 意为提出、摆姿势，均不符合语境。正确答案为B。
```

## 操作流程

### Step 1: 读取题目

通过 CDP eval 提取当前题目信息：

```javascript
(() => {
  const stem = document.querySelector('.editorContent_AOd11')?.innerText?.replace(/^字号\n/, '');
  const optBlock = document.querySelector('.editorOptionBlock_15eG-');
  const options = optBlock ? Array.from(optBlock.querySelectorAll('.editorOptions_3mI05')).map(el => {
    const letter = el.querySelector('.editorOption_2hS5L, [class*=Active]')?.innerText?.trim();
    const text = el.querySelector('.editorOptionContent_2Ygdm')?.innerText?.replace(/^字号\n/, '');
    return { letter, text };
  }) : [];
  const activeOption = optBlock?.querySelector('[class*=Active]')?.innerText?.trim();
  return { stem, options, activeOption };
})()
```

`activeOption` 字母即为正确答案（用户已标选）。

### Step 2: 生成解析内容

根据题目和正确答案，按上述格式生成三段文字。

### Step 3: 填入参考解析（核心）

**⚠️ 关键陷阱：必须通过模拟粘贴事件写入，不能用 `document.execCommand('insertText')`。**

`execCommand('insertText')` 只修改 DOM，不更新 Draft.js 内部 state。保存时 Draft.js 从自己的 state 读取，导致前面的内容丢失。必须用 `ClipboardEvent('paste')` 触发 Draft.js 的粘贴处理器，才能正确创建多行 block。

```javascript
(() => {
  const editor = document.querySelector('.editorTiBlock_1bIZv .public-DraftEditor-content');
  editor.focus();
  // 清空现有内容
  document.execCommand('selectAll', false, null);
  document.execCommand('delete', false, null);
  // 模拟粘贴事件 — 正确触发 Draft.js paste handler，创建多行 block
  const fullText = '【翻译】...\n【选项翻译】...\n【解析】...';
  const dataTransfer = new DataTransfer();
  dataTransfer.setData('text/plain', fullText);
  const pasteEvent = new ClipboardEvent('paste', {
    clipboardData: dataTransfer,
    bubbles: true,
    cancelable: true
  });
  editor.dispatchEvent(pasteEvent);
  return 'done';
})()
```

### Step 4: 保存并进入下一题

```javascript
(() => {
  const btns = document.querySelectorAll('.editorTiToolBtn__U9DT');
  for (const btn of btns) {
    if (btn.innerText.includes('创建下一题')) { btn.click(); return 'clicked'; }
  }
  return 'not found';
})()
```

### Step 5: 验证下一题加载

等待 1-2 秒后重新执行 Step 1，确认题目已切换。

## 批量答题模式

循环执行 Step 1→2→3→4→5，直到用户喊停或题库答完。

## 题号导航

点击左侧分页器可跳转到指定题号：

```javascript
(() => {
  const items = document.querySelectorAll('.paginationItem_39gcL');
  for (const item of items) {
    if (item.innerText.trim() === '1') { item.click(); return 'clicked'; }
  }
})()
```

当前题号 class 含 `paginationItemActive`。

## 站点经验

操作中积累的站点经验存储在 `references/site-patterns/shuashuati.com.md` 中，确定目标网站后加载。
