---
name: yt-cli-frame
description: |
  Vue 项目初始化，支持 PC 端和 H5 端。
  当用户说"前端项目初始化"、"前端脚手架"、"项目初始化"、"创建项目"、"新建项目"、"vue项目"、"vue3项目"、"vue2项目"、"pc项目"、"h5项目"、"移动端项目" 时使用此技能。
---

# Vue 项目初始化技能

适用于 Windows 环境（通过 bash 调用 PowerShell），支持 4 种模板：
- **PC 端**：Vue3 + @yto/ui-next、Vue2 + @yto/ui
- **H5 端**：Vue3 + Vant、Vue2 + Vant

## 核心原则

> **所有 PowerShell 命令必须先用 Write 工具写成 `.ps1` 脚本文件，再通过 `powershell -ExecutionPolicy Bypass -File` 执行。**
> 禁止在 bash 命令行中直接拼写 PowerShell 语法（`$` 变量、`Get-Random` 等会被 bash 错误解析）。


---

## 模板信息

| 模板标识 | 类型 | 技术栈 | Git sparse-checkout 路径 |
|---------|------|--------|--------------------------|
| vue3x-cli-frame | PC | Vue3 + @yto/ui-next | `vue3/vue3x-cli-frame/*` |
| vue2x-cli-frame | PC | Vue2 + @yto/ui | `vue2/vue2x-cli-frame/*` |
| vite-vue3-cli | H5 | Vue3 + Vant | `vue3/vite-vue3-cli/*` |
| vite-vue2-cli | H5 | Vue2 + Vant | `vue2/vite-vue2-cli/*` |

---

## 第一步：确认模板类型

### 1.1 解析用户意图

根据用户描述自动判断模板类型：

| 用户描述关键词 | 推荐模板 |
|--------------|--------|
| "vue3" + "pc" / 无平台说明 | vue3x-cli-frame |
| "vue2" + "pc" / 无平台说明 | vue2x-cli-frame |
| "vue3" + "h5" / "移动端" | vite-vue3-cli |
| "vue2" + "h5" / "移动端" | vite-vue2-cli |
| "h5" / "移动端"（无版本说明） | vite-vue3-cli（默认 Vue3） |
| 无任何说明 | vue3x-cli-frame（默认 PC + Vue3） |

### 1.2 确认选择

如果用户已明确指定模板类型（如"vue3 pc项目"、"vue2 h5项目"），则直接使用，无需再次询问。

否则使用 `AskUserQuestion` 工具一次性询问，提供4个选项：

```
AskUserQuestion:
  questions:
    - question: "请选择项目模板类型？"
      header: "模板类型"
      multiSelect: false
      options:
        - label: "Vue3 PC端"
          description: "PC端网页应用，使用 Vue3 + @yto/ui-next 组件库"
        - label: "Vue2 PC端"
          description: "PC端网页应用，使用 Vue2 + @yto/ui 组件库"
        - label: "Vue3 H5端"
          description: "移动端H5应用，使用 Vue3 + Vant 组件库"
        - label: "Vue2 H5端"
          description: "移动端H5应用，使用 Vue2 + Vant 组件库"
```

根据用户选择的 label 确定模板：
| 用户选择 | 模板标识 |
|---------|--------|
| "Vue3 PC端" | vue3x-cli-frame |
| "Vue2 PC端" | vue2x-cli-frame |
| "Vue3 H5端" | vite-vue3-cli |
| "Vue2 H5端" | vite-vue2-cli |

> **项目名称自动获取**：项目名称由 `.trae` 目录的父级目录名决定。例如 `.trae` 位于 `D:\projects\my-app\.trae`，则项目名称为 `my-app`。

### 1.3 确定变量

根据选择确定变量：

| 模板标识 | SPARSE_PATH | TEMPLATE_SUBDIR |
|---------|-------------|-----------------|
| vue3x-cli-frame | `vue3/vue3x-cli-frame/*` | `vue3\vue3x-cli-frame` |
| vue2x-cli-frame | `vue2/vue2x-cli-frame/*` | `vue2\vue2x-cli-frame` |
| vite-vue3-cli | `vue3/vite-vue3-cli/*` | `vue3\vite-vue3-cli` |
| vite-vue2-cli | `vue2/vite-vue2-cli/*` | `vue2\vite-vue2-cli` |

- `PROJECT_NAME`：`.trae` 父级目录的名称（即 `WORKSPACE_DIR` 的目录名）

---

## 第二步：使用 Write 工具创建初始化脚本，然后执行

**必须使用 `Write` 工具**将以下 PowerShell 脚本写入临时文件（如 `WORKSPACE_DIR\.psDir\init-project.ps1`），然后用 Bash 执行：

```
powershell -ExecutionPolicy Bypass -File "WORKSPACE_DIR\.psDir\init-project.ps1"
```

脚本内容模板（将 `WORKSPACE_DIR`、`SPARSE_PATH`、`TEMPLATE_SUBDIR` 替换为实际值后写入）：

```powershell
$ErrorActionPreference = "Stop"
$WORKSPACE_DIR = "实际工作区路径"
$TEMP_DIR = "$env:TEMP\vue-init-$(Get-Random)"

try {
    # 1. 创建临时目录并拉取模板
    New-Item -ItemType Directory -Path $TEMP_DIR | Out-Null
    Set-Location $TEMP_DIR
    git init 2>&1 | Out-Null
    git remote add origin "https://git.yto.net.cn/yto-t/TechnologyPlatform/front-framwork/iframe.git" 2>&1 | Out-Null
    git config core.sparseCheckout true
    "实际SPARSE_PATH" | Out-File -FilePath ".git\info\sparse-checkout" -Encoding ascii
    git pull origin master --depth 1 2>&1 | Out-Null

    # 2. 校验模板目录
    $TEMPLATE_DIR = "$TEMP_DIR\实际TEMPLATE_SUBDIR"
    if (-not (Test-Path $TEMPLATE_DIR)) {
        Write-Output "TEMPLATE_FAIL"
        exit 1
    }
    Write-Output "TEMPLATE_OK"

    # 3. 复制到目标目录（直接复制到 WORKSPACE_DIR）
    robocopy $TEMPLATE_DIR $WORKSPACE_DIR /E /NFL /NDL /NJH /NJS | Out-Null
    Write-Output "COPY_OK"
}
finally {
    # 4. 确保临时目录被清理
    Set-Location $WORKSPACE_DIR
    if (Test-Path $TEMP_DIR) {
        Remove-Item -Recurse -Force $TEMP_DIR -ErrorAction SilentlyContinue | Out-Null
    }
    Write-Output "DONE"
}
```

> **关键说明**：
> - 必须使用 `robocopy` 而非 `Copy-Item`，`robocopy` 能保证二进制级别精确复制。
> - 模板文件将直接复制到 `WORKSPACE_DIR`（即 `.trae` 的父级目录）。
> - 拉取代码时 PowerShell 可能将 git 的标准输出误判为错误，**以 `TEMPLATE_OK` 输出为准**，不以退出码为准。

---

## 第三步：使用 search_replace 工具替换 package.json 的 name 字段

**禁止通过 PowerShell/Node.js 脚本替换 package.json**（PowerShell here-string 中 `$1`/`$2` 等会被当作变量解析，导致 JS 脚本语法错误）。

### 正确做法：

1. **使用 `read_file` 工具读取 `WORKSPACE_DIR\package.json`**，确认原始 name 字段的完整内容

2. **使用 `search_replace` 工具进行替换**，必须确保：
   - `original_text` 包含原始 name 行的**完整内容**（包括引号）
   - `new_text` 包含新 name 值的**完整内容**（包括引号）
   - **保留 JSON 格式**，确保逗号正确

### 各模板默认 name 字段：

| 模板标识 | 原始 name 值 |
|---------|-------------|
| vue3x-cli-frame | `"name": "webpack-vue3-cli-pc"` |
| vue2x-cli-frame | 以实际读取为准 |
| vite-vue3-cli | 以实际读取为准 |
| vite-vue2-cli | 以实际读取为准 |

### 示例（vue3x-cli-frame 模板）：

```json
// 读取 package.json 后，确认原始内容为：
{
  "name": "webpack-vue3-cli-pc",
  "version": "1.0.0",
  ...
}

// 使用 search_replace 进行替换：
{
  "file_path": "WORKSPACE_DIR\\package.json",
  "replacements": [
    {
      "original_text": '"name": "webpack-vue3-cli-pc"',
      "new_text": '"name": "实际项目名"'
    }
  ]
}
```

### ⚠️ 关键注意事项：

1. **不要只替换值部分**，必须替换完整的 `"name": "xxx"` 键值对
2. **替换后必须验证 JSON 格式**，确保没有丢失逗号或引号
3. 如果原始内容有尾随逗号（如 `"name": "xxx",`），替换后的内容也必须保留逗号
4. 替换完成后，建议使用 `get_problems` 工具检查 package.json 是否有语法错误

---

## 第四步：清理临时脚本文件

使用 `DeleteFile` 工具删除第二步创建的 `init-project.ps1` 脚本（位于 `.psDir` 目录下）。

---

## 第五步：完成提示

输出以下信息告知用户：

```
========================================
  项目 'PROJECT_NAME' 初始化成功！
  模板：TEMPLATE_TYPE
========================================

项目路径: WORKSPACE_DIR

接下来可以执行：
  pnpm install   （推荐）或 npm install
  pnpm run dev
========================================
```

如果用户要求"并运行"，则继续执行：
1. `cd WORKSPACE_DIR && pnpm install`（timeout 设置 300000ms）
2. 启动 dev server 前，先检查端口占用（默认端口 8040）：
   ```
   netstat -ano | findstr ":8040"
   ```
   如果有占用，用 `powershell -Command "Stop-Process -Id PID -Force"` 释放端口
3. `cd WORKSPACE_DIR && pnpm run dev`（使用 `run_in_background: true`）

---

## 注意事项

- `WORKSPACE_DIR` 为 `.trae` 目录的父级目录，也是项目的根目录
- `PROJECT_NAME` 自动从 `WORKSPACE_DIR` 的目录名获取，无需用户输入
- **所有 PowerShell 逻辑必须通过 Write 工具写入 .ps1 文件后执行**，严禁在 bash 中直接拼写 PowerShell 语法
- **package.json 修改必须使用 Edit 工具**，严禁使用 PowerShell/Node.js 脚本（避免转义问题）
- 启动 dev server 前必须检查端口占用并处理
