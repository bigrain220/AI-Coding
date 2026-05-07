---
name: web-cli-frame
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
>
> **除非用户明确要求"并运行"，否则默认不自动执行 `pnpm install` 或 `pnpm run dev`。**
> 项目初始化完成后，仅提供手动执行命令的提示。

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

根据用户描述判断需要询问的内容：

| 用户描述关键词 | 询问策略 |
|--------------|--------|
| "vue3" + "pc" | 直接使用 vue3x-cli-frame 模板 |
| "vue3" + "h5" / "移动端" | 直接使用 vite-vue3-cli 模板 |
| "vue2" + "pc" | 直接使用 vue2x-cli-frame 模板 |
| "vue2" + "h5" / "移动端" | 直接使用 vite-vue2-cli 模板 |
| "vue3" | 询问平台类型（PC/H5） |
| "vue2" | 询问平台类型（PC/H5） |
| "pc" | 询问Vue版本（Vue3/Vue2） |
| "h5" / "移动端" | 询问Vue版本（Vue3/Vue2） |
| 其他情况 | 询问完整模板类型 |

### 1.2 确认选择

必须让用户明确选择后才执行，禁止使用默认逻辑：

1. **如果用户已明确指定完整模板类型**（如"vue3 pc项目"），则直接使用，无需再次询问。

2. **如果用户只指定了Vue3**，则询问平台类型：

```
AskUserQuestion:
  questions:
    - question: "请选择Vue3项目的平台类型？"
      options:
        - label: "PC端"
          description: "PC端网页应用，使用 Vue3 + @yto/ui-next 组件库"
        - label: "H5端"
          description: "移动端H5应用，使用 Vue3 + Vant 组件库"
```

3. **如果用户只指定了Vue2**，则询问平台类型：

```
AskUserQuestion:
  questions:
    - question: "请选择Vue2项目的平台类型？"
      options:
        - label: "PC端"
          description: "PC端网页应用，使用 Vue2 + @yto/ui 组件库"
        - label: "H5端"
          description: "移动端H5应用，使用 Vue2 + Vant 组件库"
```

4. **如果用户只指定了PC端**，则询问Vue版本：

```
AskUserQuestion:
  questions:
    - question: "请选择PC端项目的Vue版本？"
      options:
        - label: "Vue3"
          description: "使用 Vue3 + @yto/ui-next 组件库"
        - label: "Vue2"
          description: "使用 Vue2 + @yto/ui 组件库"
```

5. **如果用户只指定了H5端**，则询问Vue版本：

```
AskUserQuestion:
  questions:
    - question: "请选择H5端项目的Vue版本？"
      options:
        - label: "Vue3"
          description: "使用 Vue3 + Vant 组件库"
        - label: "Vue2"
          description: "使用 Vue2 + Vant 组件库"
```

6. **如果用户未指定任何信息**，则询问完整模板类型：

```
AskUserQuestion:
  questions:
    - question: "请选择项目模板类型？"
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

### 1.3 确定变量

| 模板标识 | SPARSE_PATH | TEMPLATE_SUBDIR |
|---------|-------------|-----------------|
| vue3x-cli-frame | `vue3/vue3x-cli-frame/*` | `vue3\vue3x-cli-frame` |
| vue2x-cli-frame | `vue2/vue2x-cli-frame/*` | `vue2\vue2x-cli-frame` |
| vite-vue3-cli | `vue3/vite-vue3-cli/*` | `vue3\vite-vue3-cli` |
| vite-vue2-cli | `vue2/vite-vue2-cli/*` | `vue2\vite-vue2-cli` |

- `PROJECT_NAME`：`.qoder` 父级目录的名称（即 `WORKSPACE_DIR` 的目录名）

---

## 第二步：执行模板拉取脚本

将以下脚本写入 `WORKSPACE_DIR\.qoder\init.ps1`，替换占位符后执行：

```powershell
# 替换：WORKSPACE_DIR / SPARSE_PATH / TEMPLATE_SUBDIR
$WS = "WORKSPACE_DIR"
$TEMP = "$env:TEMP\vue-init-$(Get-Random)"
New-Item -ItemType Directory -Path $TEMP | Out-Null
Set-Location $TEMP
git init 2>&1 | Out-Null
git remote add origin "https://git.yto.net.cn/yto-t/TechnologyPlatform/front-framwork/iframe.git" 2>&1 | Out-Null
git config core.sparseCheckout true
"SPARSE_PATH" | Out-File ".git\info\sparse-checkout" -Encoding ascii
git pull origin master --depth 1 2>&1 | Out-Null

$TPL = "$TEMP\TEMPLATE_SUBDIR"
if (-not (Test-Path $TPL)) { Write-Output "TEMPLATE_FAIL"; exit 1 }
Write-Output "TEMPLATE_OK"

robocopy $TPL $WS /E /NFL /NDL /NJH /NJS | Out-Null
Write-Output "COPY_OK"

Set-Location $WS
Remove-Item -Recurse -Force $TEMP 2>&1 | Out-Null
Write-Output "DONE"
```

执行命令：

```bash
powershell -ExecutionPolicy Bypass -File "WORKSPACE_DIR\.qoder\init.ps1"
```

> 以输出中是否含 `TEMPLATE_OK` 为准，忽略 git stderr 误报。

---

## 第三步：修改 package.json 的 name 字段

1. 用 `read_file` 读取 `WORKSPACE_DIR\package.json`，确认原始 `name` 值
2. 用 `search_replace` 替换（必须替换完整键值对，保留逗号）：

```
original_text: "name": "原始值"
new_text:      "name": "PROJECT_NAME"
```

---

## 第四步：清理并提示完成

1. 删除 `WORKSPACE_DIR\.qoder\init.ps1`

2. **默认情况下（用户未要求"并运行"）**，仅输出提示信息，不自动执行 `pnpm install` 或 `pnpm run dev`：

```
========================================
  项目 'PROJECT_NAME' 初始化成功！
  模板：TEMPLATE_TYPE
  路径：WORKSPACE_DIR

  下一步：
    pnpm install
    pnpm run dev
========================================
```

3. **仅当用户明确要求"并运行"时**，才依次执行以下操作：

   a. `pnpm install`（timeout 300000ms）
   b. 检查默认端口 8080 是否占用：`netstat -ano | findstr ":8080"`，如有则 kill
   c. `pnpm run dev`（`run_in_background: true`）

---

## 注意事项

- `WORKSPACE_DIR` 为 `.qoder` 目录的父级目录，也是项目的根目录
- `PROJECT_NAME` 自动从 `WORKSPACE_DIR` 的目录名获取，无需用户输入
- **所有 PowerShell 逻辑必须通过 Write 工具写入 `.ps1` 文件后执行**，严禁在 bash 中直接拼写 PowerShell 语法
- **package.json 修改必须使用 `search_replace` 工具**，严禁使用 PowerShell/Node.js 脚本（避免转义问题）
- 启动 dev server 前必须检查端口占用并处理