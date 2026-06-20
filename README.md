# shuashuati-skill

在刷刷题(shuashuati.com)题库编辑页面自动答题的 opencode skill。

## 功能

- 自动读取题目和选项
- 识别用户已标选的正确答案
- 生成翻译+选项翻译+解析，一次性填入"参考解析"字段
- 保存并自动进入下一题
- 支持批量答题模式

## 安装

将此 skill 放入 opencode 的 skills 目录：

```
~/.config/opencode/skills/shuashuati-quiz/
├── SKILL.md
└── references/
    └── site-patterns/
        └── shuashuati.com.md
```

## 前置条件

- opencode
- [web-access skill](https://github.com/eze-is/web-access)（用于 CDP 浏览器操作）
- Chrome / Edge 浏览器已开启远程调试

## 解析格式

```
【翻译】<题干中文翻译>
【选项翻译】A. <单词> <中文> ｜ B. <单词> <中文> ｜ C. <单词> <中文> ｜ D. <单词> <中文>
【解析】<语法/词汇解析，说明正确答案及排除理由>
```

## 技术要点

- 刷刷题使用 Draft.js 编辑器，必须通过 `ClipboardEvent('paste')` 写入内容，`document.execCommand('insertText')` 无法更新 Draft.js 内部状态
- 正确答案通过选项字母的 `Active` class 标识
- 保存按钮为"保存题目、并创建下一题"
