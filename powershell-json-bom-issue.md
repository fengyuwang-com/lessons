# PowerShell 写入 JSON 文件加 BOM 导致解析失败

## 日期
2026-07-02

## 问题
用 PowerShell 的 `Set-Content -Encoding UTF8` 写入 JSON 文件后，解析报错：
```
expected value at line 1 column 1
```

## 原因
PowerShell 的 `Set-Content -Encoding UTF8` 默认添加 BOM（字节序标记，`EF BB BF`）。
JSON 解析器不认 BOM，把前 3 个字节当成非法字符。

## 正确做法
用 .NET 方法写入，指定无 BOM：

```powershell
$json = '{"key": "value"}'
[System.IO.File]::WriteAllText(
    "C:\path\file.json",
    $json,
    [System.Text.UTF8Encoding]::new($false)
)
```

或用 `ConvertTo-Json` 后用 `WriteAllText`：

```powershell
$obj | ConvertTo-Json -Depth 10 |
    ForEach-Object {
        [System.IO.File]::WriteAllText("file.json", $_, [System.Text.UTF8Encoding]::new($false))
    }
```

## 验证
```powershell
$bytes = [System.IO.File]::ReadAllBytes("file.json")
# 第一个字节应该是 123（即 '{'），不是 239（BOM 第一字节）
```

## 教训
Windows 上写 JSON/XML 等文本文件要注意 BOM 问题。用 .NET 方法更可靠。
