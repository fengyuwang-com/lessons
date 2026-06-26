# 密钥泄露预防方案

## 本地预防（提交前拦截）

| 工具 | 安装 | 说明 |
|------|------|------|
| [pre-commit](https://pre-commit.com) | `pip install pre-commit` | 提交前钩子框架 |
| [gitleaks](https://github.com/gitleaks/gitleaks) | `brew install gitleaks` | 检测 git 中的密钥 |
| [truffleHog](https://github.com/trufflesecurity/trufflehog) | `pip install trufflehog` | 深度扫描 |
| [talisman](https://github.com/thoughtworks/talisman) | 一键安装脚本 | ThoughtWorks 出品 |

推荐组合：`pre-commit` + `gitleaks` hook

```yaml
# .pre-commit-config.yaml
repos:
  - repo: https://github.com/gitleaks/gitleaks
    rev: v8.18.0
    hooks:
      - id: gitleaks
```

## CI/CD 扫描

GitHub Actions 集成 gitleaks：

```yaml
name: secret-scan
on: [push, pull_request]
jobs:
  scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
      - uses: gitleaks/gitleaks-action@v2
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

## GitHub 内置防护

- **Secret Scanning** — 公开仓库自动启用，GitHub 会检测已知格式的密钥并通知
- **推送保护** — 开启后阻止含密钥的推送（Settings → Code security → Push protection）
