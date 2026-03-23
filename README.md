# 🎮 Runox 自动重启
使用 SeleniumBase + GitHub Actions 每 **1 小时**自动执行 Stop→Start 保持 Runox 免费服务器运行。

---

## ⚙️ 配置步骤

### 1️⃣ 准备仓库
- **公库**：存放 `.github/workflows/runox-awaken.yml`
- **私库**（`Runox-Awaken-Private`）：存放 `awaken.py`

### 2️⃣ 添加 Secrets
进入公库 → **Settings** → **Secrets and variables** → **Actions** → **New repository secret**：

| 🔑 Secret 名称 | 📝 格式 | ✅ 必填 |
|---|---|---|
| `RUNOX_ACCOUNT` | `email,password` | ✅ |
| `PRIVATE_REPO_TOKEN` | GitHub PAT（有私库读取权限） | ✅ |
| `TG_BOT` | `chat_id,bot_token` | 可选 |
| `GOST_PROXY` | `socks5://user:pass@host:port` | 可选 |

### 3️⃣ 启用 Actions
进入公库 **Actions** 标签页，点击 **Enable GitHub Actions**。

### 4️⃣ 手动触发测试
**Actions** → **Runox-Awaken** → **Run workflow**

---

## 🕐 运行时间
每 **1 小时**自动运行一次（UTC `0 * * * *`）。  
可在 `.github/workflows/runox-awaken.yml` 的 `cron` 表达式中修改。

---

## 📊 运行逻辑

脚本登录后读取服务器状态，根据不同状态执行不同操作：

### 状态一：Active / Online（服务器运行中）
服务器正在运行，每小时定期执行 Stop→Start 续期。
```
登录 → 进入服务器详情页
→ 点击 Stop
→ 等待 Start 按钮出现
→ 点击 Start
→ 等待状态变为 Online
→ 📩 TG 推送：✅ Stop→Start 成功 + 使用期限
```

### 状态二：HIBERNATED（服务器已休眠）
服务器进入休眠，先通过兜底唤醒流程恢复，再执行 Stop→Start。
```
登录 → 进入服务器详情页
→ Cloudflare Turnstile 验证
→ 点击 Start / Restore（唤醒休眠）
→ 等待 Start 按钮 → 点击 Start
→ 等待状态变为 Online
→ 📩 TG 推送：✅ 兜底唤醒成功 + 使用期限
→ 继续执行 Stop→Start
→ 📩 TG 推送：✅ Stop→Start 成功 + 使用期限
```

### 状态三：未知 / 其他
状态无法识别时，先尝试 Stop→Start，若失败则降级为兜底唤醒。
```
登录 → 进入服务器详情页
→ 尝试 Stop→Start
→ 成功：📩 TG 推送结果
→ 失败：执行兜底唤醒流程 → 📩 TG 推送结果
```

---

## 📩 TG 推送说明

| 情况 | 是否推送 | 推送内容 |
|---|---|---|
| Stop→Start 成功 | ✅ | `✅ Stop→Start 成功` + 使用期限 |
| Stop→Start 失败 | ✅ | `❌ Stop→Start 失败：Start 按钮未出现` |
| 兜底唤醒成功 | ✅ | `✅ 兜底唤醒成功` + 使用期限 |
| 兜底唤醒后状态异常 | ✅ | `⚠️ 唤醒后状态异常：xxx` |
| Turnstile 验证失败 | ✅ | `❌ 唤醒失败：Turnstile 验证未通过` |
| Start 按钮超时 | ✅ | `❌ Start 按钮未出现` |
| 登录失败 | ✅ | `❌ 登录超时` 等 |
| Manage 链接未找到 | ✅ | `❌ 未找到 Manage 链接` |

**推送格式示例：**
```
🎮 Runox 通知
🕐 运行时间: 2026-03-16 11:07:58
🖥 服务器: Runox-NL
📊 结果: ✅ Stop→Start 成功
⏱️ 使用期限：2 hours y 59 minutes
```

---

## 🖼️ 调试截图

每次运行后，截图会上传至 Actions Artifacts（`debug-screenshots`），方便排查问题：

| 截图文件 | 说明 |
|---|---|
| `dashboard.png` | 登录后 dashboard 页面 |
| `server_detail.png` | 进入服务器详情页时的状态 |
| `turnstile_click_pos.png` | CF 验证点击位置（红点标记） |
| `turnstile_fail.png` | CF 验证失败时的页面 |
| `start_btn_timeout.png` | Start 按钮等待超时时的页面 |
| `final_status.png` | 启动后的最终状态页面 |
