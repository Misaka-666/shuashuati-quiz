---
domain: shuashuati.com
aliases: [刷刷题, shuashuati]
updated: 2026-06-20
---

## 平台特征
- 题库编辑器使用 Draft.js rich text editor
- 所有文本编辑区都是 contenteditable div，class 含 `public-DraftEditor-content`
- 正确答案选项字母 class 含 `editorOptionActive`，其余为 `editorOption`

## 有效模式

### DOM 选择器
| 元素 | 选择器 |
|------|--------|
| 题干 | `.editorContent_AOd11 .public-DraftEditor-content` |
| 选项块 | `.editorOptionBlock_15eG-` |
| 单个选项 | `.editorOptions_3mI05` |
| 选项字母 | `.editorOption_2hS5L` 或 `[class*=Active]` |
| 选项文字 | `.editorOptionContent_2Ygdm .public-DraftEditor-content` |
| 参考解析 | `.editorTiBlock_1bIZv .public-DraftEditor-content` |
| 保存按钮 | `.editorTiToolBtn__U9DT`（含"创建下一题"） |
| 题号分页 | `.paginationItem_39gcL` |

### 编辑器操作
- 读取文字：`element.innerText`（去掉前缀 `字号\n`）
- 写入文字：focus → selectAll → delete → **模拟粘贴事件**（`ClipboardEvent('paste')` + `DataTransfer`）

写入代码模板：
```javascript
const editor = document.querySelector('.editorTiBlock_1bIZv .public-DraftEditor-content');
editor.focus();
document.execCommand('selectAll', false, null);
document.execCommand('delete', false, null);
const dataTransfer = new DataTransfer();
dataTransfer.setData('text/plain', fullText);
editor.dispatchEvent(new ClipboardEvent('paste', { clipboardData: dataTransfer, bubbles: true, cancelable: true }));
```

## 已知陷阱
- **execCommand 不更新 Draft.js state**（2026-06-20 发现）：`document.execCommand('insertText')` 只修改 DOM，不更新 Draft.js 内部 state。保存时 Draft.js 从自己的 state 读取，导致前面的内容丢失。必须用 `ClipboardEvent('paste')` 触发 Draft.js 的粘贴处理器
- **手动粘贴可行**：用户手动 Ctrl+V 粘贴能完整保留多行内容，因为走的是 Draft.js 的 paste handler
- **innerText 前缀**：编辑器内容区 innerText 开头带 `字号\n`，需 strip
- 选项字母 `activeOption` 可通过 `[class*=Active]` 选择器直接获取，无需手动判断
