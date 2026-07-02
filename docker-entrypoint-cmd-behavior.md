# Docker Entry point / CMD 行为误解

## 日期
2026-07-02

## 问题
用 `docker run` 传 cmd 参数想覆盖配置，但容器启动后还是用了旧配置。

## 原因
Docker 的 entrypoint 和 cmd 关系：
- 有 entrypoint 时，cmd 是传给 entrypoint 的参数，不是直接执行的命令
- entrypoint.sh 里用 `PROVIDER` 环境变量选配置，完全忽略了 cmd 参数

```bash
# 这样写不会生效
docker run moonbridge:local /app/moonbridge -config /app/config.yml

# 实际执行的是 entrypoint.sh，它自己决定了用哪个配置
```

## 正确做法
用 `--entrypoint` 绕过 entrypoint.sh：

```bash
docker run -d \
  --name moonbridge \
  --entrypoint /app/moonbridge \
  moonbridge:local \
  -config /app/config.yml \
  -addr 0.0.0.0:38440
```

## 教训
改 Docker 容器配置前，先检查 entrypoint 脚本的逻辑：
```bash
docker exec <container> cat /app/entrypoint.sh
```
