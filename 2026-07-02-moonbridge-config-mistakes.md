# 2026-07-02 Moon Bridge 配置错误总结

## 任务
将 Moon Bridge + Codex 的 API 从 DeepSeek 原厂切换到 opencode.ai 中转。

## 犯过的错误

### 错误 1: 改错配置文件
一开始改的是 OpenClaw 的配置，不是 Moon Bridge。没有先搞清楚用户说的是哪个工具。

**教训**: 先问清楚，再动手。

### 错误 2: 不知道 Moon Bridge 是 Docker 服务
用户说"moon bridge"，我搜了半天本地文件才发现是 Docker 容器。

**教训**: 先问部署方式（本地二进制 / Docker / 其他）。

### 错误 3: 没有检查 entrypoint.sh
容器的 entrypoint.sh 会根据 `PROVIDER` 环境变量选择配置文件，而且**忽略 cmd 参数**。我用 `docker run` 传了 cmd 参数，但 entrypoint 完全没用到。

```bash
# 错误：以为这样能覆盖配置
docker run -d --name moonbridge moonbridge:local /app/moonbridge -config /app/config-opencode.yml

# 实际执行的是 entrypoint.sh，它选了 config-deepseek.yml
```

**教训**: Docker 容器有 entrypoint 时，cmd 是传给 entrypoint 的参数，不是直接执行的命令。要先检查 entrypoint 的逻辑。

### 错误 4: 删除容器内文件后重建容器，文件又回来了
我在运行中的容器里删除了 config-deepseek.yml，但重建容器后文件又出现了（从镜像层恢复）。

**教训**: 容器内的修改不持久化。要改配置必须：
1. 用 `-v` 挂载外部文件
2. 或者修改 Dockerfile 重新构建镜像

### 错误 5: 没有验证 provider 名称
改完配置后只测试了"能不能通"，没有检查日志里的 `provider=` 字段。实际上还在用 deepseek provider，不是 opencode。

**教训**: 改完配置必须检查日志确认生效：
```bash
docker logs moonbridge --tail 5
# 要看到 provider=opencode，不是 provider=deepseek
```

### 错误 6: PowerShell 写入 JSON 加了 BOM
用 PowerShell 的 `Set-Content -Encoding UTF8` 写入 JSON 文件时，默认加了 BOM（字节序标记），导致 JSON 解析器报错 "expected value at line 1 column 1"。

**教训**: 写 JSON 文件用 `[System.IO.File]::WriteAllText()` 并指定 `UTF8Encoding::new($false)` 去掉 BOM。

## 正确的配置流程

### 1. 创建配置文件
在本地创建 `config-{provider}.yml`，挂载到容器。

### 2. 启动容器（关键）
```bash
docker run -d \
  --name moonbridge \
  -p 127.0.0.1:38440:38440 \
  --entrypoint /app/moonbridge \
  -v "/path/to/config.yml:/app/config.yml" \
  moonbridge:local \
  -config /app/config.yml \
  -addr 0.0.0.0:38440
```

**必须用 `--entrypoint /app/moonbridge`** 绕过 entrypoint.sh。

### 3. 验证
```bash
docker logs moonbridge --tail 5
# 确认 provider=xxx 是你配置的名称
```

## 根本原因
对 Docker 的 entrypoint/cmd 机制不熟悉，没有在改配置前完整阅读现有的脚本和配置逻辑。
