# Lessons Learned: API Key Leak Incident

## 事件回顾

2026年6月，发现 opencode Go key 被他人使用。经排查，泄露源为公开仓库 `llm-text-processor` 中的 `backend/config.json` 文件，其中硬编码了一个有效的 opencode API key。

该仓库是公开的，key 被提交到 git 历史后立即暴露在互联网上。

## 泄露原因

1. **密钥硬编码在配置文件中** — API key 直接写在 `config.json`，而非通过环境变量注入
2. **配置文件被 git 跟踪** — `config.json` 没有加入 `.gitignore`，每次提交都包含密钥
3. **仓库为公开** — 任何人都可以访问和克隆
4. **无预提交检查** — 没有使用 gitleaks / pre-commit 等工具在提交前扫描密钥

## 教训

### 密钥管理

- **永远不要**将 API key、token、密码等凭据硬编码在源码或配置文件中
- 使用环境变量（`process.env.OPENAI_API_KEY`）或专门的密钥管理服务（如 1Password CLI、AWS Secrets Manager）
- 配置文件模板（如 `config.example.json`）应使用占位符，实际配置文件通过 `.gitignore` 排除

### 仓库安全

- 公开仓库提交前，确认没有凭据、内网地址、调试信息被包含
- `.gitignore` 从一开始就配置完善，包括常见的配置文件、`.env`、密钥文件等
- 敏感文件即使后来加入 `.gitignore`，git 历史中仍存在记录，需要重写历史才能彻底清除

### git 历史

- 一旦密钥被提交到 git，即使后续删除文件，历史中仍有记录
- 使用 `git filter-branch` 或 `git filter-repo` 可以重写历史移除密钥，但需要 force push，会影响协作者
- 最佳方案：提交前预防

### 事后处理流程

1. **立即吊销泄露的 key**（最重要的一步）
2. 生成新 key 替换
3. 重写 git 历史清除密钥记录
4. force push 到远程
5. 加固仓库配置，防止再次发生

## 预防措施

见仓库根目录 [SECURITY.md](SECURITY.md) 中推荐的工具和流程。

---

*记录于 2026年6月*
