# Git 历史中移除密钥

## 日期
2026-06

## 问题
API key 已经被提交到 git 历史，即使删除文件，历史中仍有记录。

## 原因
git 保存完整历史。删除文件只是在最新 commit 中移除，历史 commit 仍包含该文件。

## 正确做法

### 1. 立即吊销 key（最重要）
去 API 提供商后台吊销泄露的 key，生成新的。

### 2. 重写 git 历史
```bash
# 安装 git-filter-repo
pip install git-filter-repo

# 移除指定文件
git filter-repo --path backend/config.json --invert-paths

# 强制推送
git push --force
```

### 3. 通知协作者
force push 后，协作者需要重新 clone 仓库。

## 教训
事后补救代价很大。最好在提交前预防（用 gitleaks 或 pre-commit hook）。
