# 公开仓库安全检查

## 日期
2026-06

## 问题
公开仓库中包含敏感信息（API key、内网地址、调试信息）。

## 原因
提交前没有检查公开仓库会暴露什么。

## 正确做法

### 1. 提交前检查
```bash
# 检查是否有敏感信息
gitleaks detect

# 或手动检查
git diff --cached
```

### 2. .gitignore 配置
```gitignore
# 配置文件
config.json
.env
.env.local

# 密钥文件
*.pem
*.key
```

### 3. 定期扫描
```bash
# 定期扫描 git 历史
gitleaks detect --log-opts="--all"
```

## 教训
公开仓库 = 互联网可见。提交前想清楚：这段代码/配置能公开吗？
