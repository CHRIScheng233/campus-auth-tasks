# submit-task — 提交任务到 Campus-Auth 仓库

## 流程概览

安全审查 → 格式修正 → 放入 `temp/`（未审核）→ 确认后移至 `tasks/` → 更新 `index.json` → 提交

## 适用场景

- 你编写了一个校园网认证任务，想提交到本仓库
- 使用 AI 辅助完成提交流程，自动完成安全审查、格式修正、索引更新

## 提交流程

### Step 1: 准备任务 JSON

从 Campus-Auth 导出或自行编写任务 JSON 文件。

### Step 2: 安全审查（⚠️ 重点）

`eval` 和 `custom_js` 类型的步骤会直接执行 JavaScript。审查每个 `script` 字段：

| 风险行为 | 说明 | 处理 |
|----------|------|------|
| 远程代码加载 | `fetch()`、`XMLHttpRequest`、`$.getScript()`、`import()`、动态创建 `<script>` 加载远程 JS | ❌ 拒绝 |
| 数据外传 | `navigator.sendBeacon()`、`fetch()`/`XMLHttpRequest` 向第三方域名发数据 | ❌ 拒绝 |
| 敏感泄露 | 读取 `localStorage`、`sessionStorage`、`document.cookie` 并外传 | ❌ 拒绝 |
| 页面跳转 | `window.open()`、`location.href=` 跳转到非认证域名 | ⚠️ 需说明 |
| 安全操作 | DOM 查询、表单填写、点击事件、文本匹配判断 | ✅ 通过 |

**安全脚本通常只做三件事：** 查找页面元素 → 填写值/触发事件 → 读取文本判断结果。

### Step 3: 格式修正

| 规则 | 说明 |
|------|------|
| 缩进 | 统一 2 空格 |
| `code` → `script` | 步骤中的 `code` 字段改名为 `script` |
| `url` 字段 | 必须为 `"{{LOGIN_URL}}"` 或省略，禁止硬编码地址 |
| `on_failure.screenshot` | 确保为 `true` |

### Step 4: 放入 `temp/`（未审核状态）

生成文件名（小写 + 下划线），放入 `temp/` 目录：

```powershell
Move-Item -Path "task.json" -Destination "temp/beijing_university.json"
```

`temp/` = 待审核，`tasks/` = 已收录。

### Step 5: 更新 `index.json`

在 `index.json` 数组末尾添加条目，`url` 指向 `tasks/` 中的文件：

```json
{
  "id": "xxx_university",
  "name": "XXX大学登录",
  "description": "适用于 XXX 大学校园网认证页面",
  "tags": ["XXX大学", "Portal"],
  "author": "your-github-username",
  "version": "1.0.0",
  "url": "https://raw.githubusercontent.com/Misyra/campus-auth-tasks/master/tasks/xxx_university.json"
}
```

### Step 6: 移至 `tasks/`

确认审查通过后从 `temp/` 移到 `tasks/`：

```powershell
Move-Item -Path "temp/xxx_university.json" -Destination "tasks/xxx_university.json"
```

### Step 7: 提交

```powershell
git add temp/xxx_university.json tasks/xxx_university.json index.json doc/
git commit -m "feat: 添加 XXX 大学登录任务"
git push
```

## 验证清单

- [ ] JSON 语法正确
- [ ] 所有 `eval`/`custom_js` 脚本已审查、无风险
- [ ] `url` 为 `"{{LOGIN_URL}}"` 或省略
- [ ] `index.json` 是合法 JSON
- [ ] 文件名与 `index.json` 的 `url` 一致
- [ ] `temp/` 和 `tasks/` 下都有对应文件
- [ ] `doc/` 目录已包含在提交中
