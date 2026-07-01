# Lessons Learned: Homepage Restructure — Punchline Misalignment & File Corruption

## 事件回顾

2026年7月，在对个人网站 fengyuwang.com 进行首页改版的过程中，出现了两次严重失误：

### 失误一：punchline 放错位置

用户要求为 portfolio（技术）页面写 5 句 punchline（核心观点短句）。经过多轮讨论，最终确定为：

- 不问对，不开始
- 系统即复利
- 一维即偏见
- 工具从问题
- 交付为开始

我错误地将这 5 句放到了 section 的 block-subtitle（副标题）位置，而用户的意思是这 5 句是**大标题**（h2 / 卡片标题）。同时用户要求"卡片描述不要改"，我也理解反了——用户说的是卡片描述保持不动，而我把 punchline 放到副标题、把卡片标题保持了原样。

**根因**：没有在动手前确认"你说的标题是指哪个 HTML 元素"。用户说"标题"，我默认是副标题（block-subtitle），但用户指的是卡片 h3 和 section h2。

### 失误二：文件被脚本重复内容损坏

在重构首页 zh-cn/index.html 时，使用 Python 脚本从 git 历史提取了旧版内容并拼接回当前文件，导致文件出现重复的 `<head>` 和 `<body>` 结构：

- 文件变成了：`<html><head>...</head><body>...<style>...</style></head><body>...` 的畸形结构
- 浏览器解析时页面内容完全错乱
- 修复过程又反复截断，导致 blog 卡片区丢失、footer 丢失

**根因**：没有理解 git 原始文件的结构就盲目拼接。`git show HEAD:file` 输出的是完整文件（含 head + body），而我只需要其中的 body 片段。

## 教训

### 沟通确认

- 用户说"标题"——先确认是 h1/h2/h3 还是副标题/卡片描述
- 用户说"卡片"——确认是指 card-grid 中的卡片还是 section 中的内容卡片
- 对模糊的描述，先画一个简单结构图确认再动手

### 文件操作

- **永远不要**用脚本从 git 历史中提取片段拼接到当前文件，除非完全理解两个版本的结构差异
- 对正在编辑的文件，每次重大修改前先 `git commit`（已做 ✅），这样出错了可以 clean checkout
- 修改后立即验证：检查 body 标签数量、关键 section 是否完整
- Python 操作 HTML 文件时，使用精确的字符串匹配，不要依赖正则表达式处理复杂的嵌套结构

### HTML 结构验证清单

修改 HTML 文件后，检查：

```bash
grep -c '<body>' file.html    # 应该 = 1
grep -c '</body>' file.html   # 应该 = 1
grep -c '<html' file.html     # 应该 = 1（或 2：开+关）
```

对于关键 section，确认它们按正确顺序出现且不重复。

## 事后处理流程

1. `git checkout HEAD -- file.html` — 恢复到最后一次 commit 的状态
2. 重新分析需要修改的部分，只做精确的字符串替换
3. 每次替换后检查文件完整性
4. 对于复杂重构，先手动在小文件上测试脚本逻辑

## 预防措施

- 涉及文件拼接的操作，先在测试文件上运行脚本
- 使用 `git stash` 保存工作区，测试通过后再 apply
- 所有 Python 文件操作脚本使用绝对路径，避免 cwd 问题
- 脚本运行后立即进行结构验证

---

*记录于 2026年7月*
