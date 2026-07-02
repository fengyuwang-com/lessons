# Lessons 规范

## 文件命名
```
{YYYY-MM-DD}-{简短英文主题}.md
```

示例：
- `docker-entrypoint-cmd-behavior.md`
- `powershell-json-bom-issue.md`
- `git-force-push-recovery.md`

## 每个文件一个独立话题
不要把多个不相关的问题写在同一个文件里。

反例：
```
2026-07-02-moonbridge-config-mistakes.md  ❌（包含 Docker、PowerShell、配置验证等多个话题）
```

正例：
```
docker-entrypoint-cmd-behavior.md         ✅
docker-container-file-persistence.md     ✅
powershell-json-bom-issue.md             ✅
verify-config-actually-applied.md        ✅
```

## 文件结构

```markdown
# {标题}

## 日期
YYYY-MM-DD

## 问题
一句话描述问题现象。

## 原因
为什么会出这个问题。

## 正确做法
怎么做才对。

## 教训
一句话总结核心教训。
```

## 不需要包含
- 不要写"我"或"你"，直接描述问题
- 不要写太长的背景介绍
- 不要写已经解决的中间步骤
