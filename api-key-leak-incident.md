# API Key 泄露事件

## 日期
2026-06

## 问题
opencode Go key 被他人使用。泄露源为公开仓库 `llm-text-processor` 中的 `backend/config.json`，硬编码了一个有效的 API key。

## 原因
1. API key 硬编码在配置文件中
2. 配置文件被 git 跟踪（没有 .gitignore）
3. 仓库是公开的
4. 没有预提交检查

## 正确做法
见 `api-key-management.md` 和 `git-history-secret-removal.md`。

## 教训
密钥永远不要硬编码在代码里。用环境变量或密钥管理服务。
