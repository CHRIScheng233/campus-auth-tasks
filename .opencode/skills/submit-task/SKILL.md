# submit-task — 提交任务到 Campus-Auth 仓库

## Description

审核并提交 Campus-Auth 任务 JSON 到仓库。完整流程：安全审查 → 格式修正 → 放入 `temp/`（未审核）→ 确认后移至 `tasks/` → 更新 `index.json` 索引 → 连带 `doc/` 一起提交。

## When to Use

- 用户拿到一个 Campus-Auth 任务 JSON 文件，想提交到任务仓库
- 需要确保任务安全、格式正确、索引完备

## Workflow

### Step 1: 读取任务 JSON 文件

读取用户提供的 JSON 文件（或从剪贴板/对话中获取内容），了解任务的基本信息：
- `name` — 任务名称
- `description` — 描述
- `metadata.author` — 作者（如存在）
- 步骤列表

### Step 2: 安全审查（⚠️ 重点）

按以下清单逐项检查，**发现问题先向用户报告，要求用户确认后再继续**：

#### 2.1 代码注入风险

`eval` 和 `custom_js` 类型的步骤会直接在当前页面上下文中执行 JavaScript。审查每个 `script` 字段：

| 风险 | 说明 | 处理方式 |
|------|------|----------|
| **远程代码加载** | 包含 `fetch()`、`XMLHttpRequest`、`$.getScript()`、`import()`、`document.createElement('script')` 等动态加载远程 JS 的行为 | ❌ 拒绝 |
| **数据外传** | 包含 `navigator.sendBeacon()`、`fetch()` 或 `XMLHttpRequest` 向第三方域名发送数据 | ❌ 拒绝 |
| **敏感信息泄露** | 尝试读取 `localStorage`、`sessionStorage`、`document.cookie` 并发送 | ❌ 拒绝 |
| **页面操控** | 包含 `window.open()`、`location.href=` 跳转到非认证域名 | ⚠️ 要求说明 |
| **无风险操作** | DOM 查询（`querySelector`、`getElementById`）、表单填写、点击事件、简单的文本匹配判断 | ✅ 通过 |

**判定原则：** 校园网认证页面的 JS 脚本通常只做三件事：
- 查找页面元素（输入框、按钮）
- 填写值、触发事件
- 读取页面文本判断登录结果

超出这个范围的操作需要用户解释原因。

#### 2.2 URL 硬编码检查

`url` 字段必须是 `"{{LOGIN_URL}}"` 或留空。**禁止**硬编码具体的认证地址：

```json
// ❌ 错误 — 硬编码地址
"url": "http://192.168.1.1"
"url": "https://auth.example.edu.cn"

// ✅ 正确 — 由用户自己设置
"url": "{{LOGIN_URL}}"
// 或
// 不写 url 字段（由系统环境变量决定）
```

#### 2.3 无意义/危险步骤检查

- 步骤是否实际有效，没有死循环或不可能满足的等待条件
- `selector` 选择器是否指向实际存在的元素
- `sleep` 步骤的 `duration` 是否合理（不超过 30000ms）
- `timeout` 是否合理（不超过 60000ms）

### Step 3: 格式修正

在不改变语义的前提下修正格式问题：

| 规则 | 说明 |
|------|------|
| **缩进** | 统一使用 2 空格缩进 |
| **尾逗号** | 移除 JSON 中多余的尾逗号 |
| **`code` → `script`** | 如果步骤同时使用 `code` 和 `script`，移除已废弃的 `code` 字段；如只有 `code` 没有 `script`，重命名为 `script` |
| **`url` 字段** | 确保为 `"{{LOGIN_URL}}"` 或省略 |
| **`on_failure.screenshot`** | 确保为 `true`（方便调试） |
| **字段顺序** | 建议按规范顺序：`name` → `description` → `metadata` → `url` → `timeout` → `variables` → `steps` → `success_conditions` → `on_success` → `on_failure` |
| **步骤 ID** | 建议使用 `s1`、`s2` 或带语义的 `fill_username`、`click_login` 等命名 |

使用以下命令修正 JSON 格式：

```powershell
python -c "import json; d=json.load(open('task.json','r',encoding='utf-8')); json.dump(d,open('task_fixed.json','w',encoding='utf-8'),ensure_ascii=False,indent=2)"
```

### Step 4: 将任务放入 temp/（未审核状态）

根据任务信息生成文件名：

- 优先使用 `metadata.author` 或 `metadata.school` 生成
- 回退到 `name` 字段
- 文件名格式：小写英文字母 + 下划线，如 `beijing_university.json`
- 规则：只保留字母、数字、下划线，其余字符替换为下划线

```python
import re
base = (author or school or name).lower().replace(' ', '_')
filename = re.sub(r'[^a-z0-9_]', '_', base) + '.json'
```

将格式化后的文件保存到 `temp/` 目录：

```powershell
Move-Item -Path "path/to/task_fixed.json" -Destination "temp/{filename}"
```

`temp/` 目录存放**未审核**的任务，`tasks/` 目录存放**已审核通过**的正式任务。

### Step 5: 更新 index.json

将任务添加到 `index.json` 数组末尾。条目的字段说明：

| 字段 | 必填 | 说明 |
|------|------|------|
| `id` | 是 | 小写字母开头的字母数字下划线组合，建议与文件名同名（不含 `.json`） |
| `name` | 是 | 任务名称（与 `name` 字段一致） |
| `description` | 是 | 任务描述 |
| `tags` | 是 | 标签数组，至少包含学校名或认证系统类型 |
| `author` | 是 | 作者 GitHub 用户名 |
| `version` | 是 | 版本号，默认为 `"1.0.0"` |
| `url` | 是 | 指向 GitHub 原始文件的 URL |

**URL 格式：**
```
https://raw.githubusercontent.com/Misyra/campus-auth-tasks/master/tasks/{filename}
```

**插入位置：** 在数组最后一项的 `}` 后面加上 `,`，然后追加新条目，确保是合法的 JSON 数组。

**读取 index.json 后验证 JSON 合法性。**

示例新增条目：

```json
{
  "id": "xxx_university",
  "name": "XXX大学登录",
  "description": "适用于 XXX 大学校园网认证页面",
  "tags": ["XXX大学", "Portal"],
  "author": "github-username",
  "version": "1.0.0",
  "url": "https://raw.githubusercontent.com/Misyra/campus-auth-tasks/master/tasks/xxx_university.json"
}
```

**验证：** 修改后运行 `python -c "import json; json.load(open('index.json','r',encoding='utf-8'))"` 确保 JSON 语法正确。

### Step 6: 从 temp/ 移入 tasks/

确认安全审查通过后，将任务从 `temp/` 移至 `tasks/`：

```powershell
Move-Item -Path "temp/{filename}" -Destination "tasks/{filename}"
```

此时 `tasks/` 下的文件才是正式收录的任务。

### Step 7: 提交

```powershell
git add temp/{filename} tasks/{filename} index.json doc/
git status   # 确认只有预期文件被暂存
git commit -m "feat: 添加 {name} 登录任务"
git push
```

**注意：** `temp/` 中的未审核文件和 `tasks/` 中的正式文件都需提交（`temp/` 保留提交记录便于追溯），`doc/` 目录随任务一起提交。

## Verification

提交前必须确认：

- [ ] JSON 语法正确（`python -c` 验证）
- [ ] 所有 `eval`/`custom_js` 的 `script` 已审查且无风险
- [ ] `url` 字段为 `"{{LOGIN_URL}}"` 或省略
- [ ] `index.json` 是合法的 JSON 数组
- [ ] 任务文件名与 `index.json` 中的 `url` 字段一致
- [ ] `temp/` 和 `tasks/` 下都有对应文件
- [ ] `doc/` 目录已包含在提交中
- [ ] 已向用户报告安全审查结果并得到确认

## Example

完整流程示例：

```
用户: 帮我提交这个北京大学的校园网认证任务

Agent: 读取任务JSON → 安全审查 → 格式修正 →
       保存到 temp/pku.json → 更新 index.json →
       从 temp/ 移至 tasks/pku.json →
       git add + commit + push
```
