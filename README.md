# Campus-Auth 任务仓库

Campus-Auth 的校园网登录任务共享仓库。

## 使用方式

在 Campus-Auth 的任务管理页面，点击 **从仓库导入** 即可浏览和安装任务。

## 任务列表

| 任务 | 说明 |
|------|------|
| **通用登录** | 语义识别表单，兼容大多数认证页面 |
| **DR.COM 模板** | 适用于 DR.COM / 深澜认证系统 |
| **选择器模板** | CSS 选择器定位，适合已知页面结构 |

## 贡献

欢迎提交 PR 添加你学校的登录任务：

1. Fork 本仓库
2. 在 `tasks/` 目录下添加你的任务 JSON 文件
3. 在 `index.json` 中添加对应条目
4. 提交 PR

### 任务 JSON 格式

```json
{
  "name": "XXX大学登录",
  "description": "适用于 XXX 大学校园网认证页面",
  "url": "https://auth.xxx.edu.cn",
  "timeout": 20000,
  "variables": {
    "username": "{{USERNAME}}",
    "password": "{{PASSWORD}}",
    "isp": "{{ISP}}"
  },
  "steps": [
    {
      "id": "fill_username",
      "type": "input",
      "description": "填写账号",
      "selector": "#username",
      "value": "{{username}}"
    }
  ],
  "success_conditions": [...],
  "on_success": { "message": "登录成功" },
  "on_failure": { "message": "登录失败", "screenshot": true }
}
```

### 步骤类型

| 类型 | 说明 |
|------|------|
| `input` | 填写输入框 |
| `click` | 点击元素 |
| `select` | 选择下拉框 |
| `wait` | 等待元素出现 |
| `wait_url` | 等待 URL 变化 |
| `eval` | 执行 JavaScript 并存储结果 |
| `custom_js` | 执行自定义 JavaScript |
| `screenshot` | 截图 |
| `sleep` | 等待指定时间 |
| `ocr` | 验证码识别 |

### 任务录制器

推荐使用 [Campus-Auth 任务录制器](https://github.com/Misyra/Campus-Auth/blob/main/development/task-recorder.user.js)（油猴脚本）可视化选取元素并生成任务 JSON。

## License

MIT
