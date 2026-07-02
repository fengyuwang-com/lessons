# API Key 管理规范

## 日期
2026-06

## 问题
API key 硬编码在 config.json 中，导致泄露。

## 原因
没有使用环境变量或密钥管理服务，直接把 key 写在代码里。

## 正确做法

### 1. 不要硬编码
```json
// ❌ 错误
{"apiKey": "sk-1234567890"}

// ✅ 正确
{"apiKey": "${OPENAI_API_KEY}"}
```

### 2. 使用环境变量
```javascript
const apiKey = process.env.OPENAI_API_KEY;
```

### 3. 使用密钥管理服务
- 1Password CLI
- AWS Secrets Manager
- HashiCorp Vault

### 4. 配置文件模板
```bash
# config.example.json（提交到 git）
{"apiKey": "YOUR_API_KEY_HERE"}

# config.json（加入 .gitignore）
{"apiKey": "sk-1234567890"}
```

## 教训
密钥是资产，不是配置。用管理资产的方式管理它。
