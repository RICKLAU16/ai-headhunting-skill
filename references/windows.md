# Windows/PowerShell 环境适配指南

_适用环境：Windows 10/11 + PowerShell 5.1/7.x_

---

## 1. PowerShell 下 `eval` 的分号截断

**问题：** PowerShell 中执行 `agent-browser eval "xxx; yyy"` 时，分号后面的内容被截断，导致 SyntaxError。

**解决方案：用 PowerShell 变量读取文件再传参**

```powershell
# 步骤 1：将 JS 代码保存为 .js 文件（UTF-8 编码）
# 步骤 2：用 PowerShell 变量读取文件内容
$js = [System.IO.File]::ReadAllText("extract_liepin.js", [System.Text.Encoding]::UTF8)
# 步骤 3：传给 eval 执行
agent-browser eval $js
```

**⚠️ eval 不支持文件路径参数：**
- ❌ `agent-browser eval -f script.js` — 报语法错误
- ❌ `agent-browser eval script.js` — 报语法错误
- ✅ 必须用 PowerShell 变量中转

---

## 2. `click @ref` 必须引号包裹

**问题：** `agent-browser click @e13` 报 "Missing arguments for: click"

**解决：** ref 参数必须用引号包裹

```powershell
# ✅ 正确
agent-browser click "@e13"
# ❌ 错误
agent-browser click @e13
```

---

## 3. 路径格式

- **JS 文件中：** 使用正斜杠（`/`）或双反斜杠（`\\`）
- **PowerShell 命令中：** 正斜杠和反斜杠均可
- **文件读取时：** 两种斜杠均可，但推荐正斜杠避免转义问题

```powershell
# 两种写法都可以
$js = [System.IO.File]::ReadAllText("d:/WorkBuddy-data/extract.js", [System.Text.Encoding]::UTF8)
$js = [System.IO.File]::ReadAllText("d:\WorkBuddy-data\extract.js", [System.Text.Encoding]::UTF8)
```

---

## 4. Python 检测命令

```powershell
# Windows 下检测 Python
Get-Command python -ErrorAction SilentlyContinue | Select-Object Source
# 或
where.exe python
```

---

## 5. 内置提取脚本的标准化部署

所有 JS 提取脚本应预置为独立 .js 文件，放在工作目录下，而非在命令行中内联含分号的代码。

**推荐的脚本命名：**
- `extract_liepin.js` — 猎聘候选人批量提取
- `extract_51job.js` — 前程无忧候选人提取
- `search_input.js` — 搜索框输入（nativeInputValueSetter）
- `click_search.js` — 搜索按钮点击

**执行方式：**
```powershell
$js = [System.IO.File]::ReadAllText("extract_liepin.js", [System.Text.Encoding]::UTF8)
agent-browser eval $js
```
