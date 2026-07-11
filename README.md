# Godlike 自动续期脚本

自动登录 Godlike 面板并续期 90 分钟。支持多账号、Cookie 自动维护、多种代理协议，并通过 Telegram 发送带截图的成功/失败通知。

> 注册地址：[https://godlike.host/cart-free](https://godlike.host/cart-free)  
> ⚠️ 忘记密码？在此重置：[https://panel.godlike.host/auth/password](https://panel.godlike.host/auth/password)

## ✨ 特性

- 🔐 **邮箱/密码登录** — 自动登录 Godlike 面板
- 👥 **多账号支持** — 最多 5 个账号（`GODLIKE_1` ~ `GODLIKE_5`）
- 🍪 **Cookie 自动缓存** — 登录成功后自动回写 Cookie 到 Secrets，后续优先使用 Cookie
- 🌐 **全协议代理** — VLESS / VMess / Trojan / Shadowsocks / SOCKS5
- 📲 **Telegram 通知** — 续期成功/失败/24h上限/冷却期都会推送带截图的消息
- ⏰ **定时任务** — GitHub Actions 每 3 小时自动运行
- 🛡️ **24h上限/冷却期识别** — 自动识别并跳过

---

## 🚀 配置与使用

### 1. 配置 Secrets

在仓库 `Settings` → `Secrets and variables` → `Actions` → `New repository secret` 中添加：

| Secret 名称 | 必填 | 说明 |
|-------------|:---:|------|
| `GODLIKE_1` | ✅ | 主账号，格式 `邮箱-----密码` |
| `GODLIKE_2` ~ `GODLIKE_5` | ❌ | 额外账号（最多到5） |
| `REPO_TOKEN` | ✅ | 用于自动回写 Cookie 的 GitHub PAT |
| `TG_BOT_TOKEN` | ❌ | Telegram Bot Token |
| `TG_CHAT_ID` | ❌ | 接收通知的 Chat ID |
| `PROXY_NODE` | ❌ | 代理节点链接（可选） |

账号 Secret 首次只需提供邮箱和密码，脚本会在登录成功后自动将 Cookie 追加到 Secret 末尾，后续优先使用 Cookie。

---

### 2. 各 Secret 详细获取方法

#### 🔑 `GODLIKE_1` ~ `GODLIKE_5`（Godlike 账号）

**格式**：`邮箱-----密码`（邮箱和密码之间是 5 个减号 `-`）

**示例**：
```
admin@example.com-----MyPassword123
```

**获取步骤**：
1. 注册 Godlike 账号：[https://godlike.host/cart-free](https://godlike.host/cart-free)
2. 如果忘记密码，重置：[https://panel.godlike.host/auth/password](https://panel.godlike.host/auth/password)
3. 在 GitHub Secret 的 Value 里填入：`你的邮箱-----你的密码`
4. 多账号配置 `GODLIKE_2`、`GODLIKE_3`...直到 `GODLIKE_5`，格式相同

**⚠️ 注意**：
- 邮箱和密码之间必须是 **5 个减号** `-`，不是 1 个
- 如果密码里包含 `-`，不影响（脚本只按 `-----` 分割）

---

#### 🔑 `REPO_TOKEN`（GitHub Personal Access Token）

**用途**：脚本登录成功后，用这个 Token 自动把 Cookie 回写到 GitHub Secrets（避免每次都走登录流程）

**获取步骤**：
1. 打开 https://github.com/settings/tokens/new
2. 配置：
   - **Note**：`Godlike Cookie 回写`（任意名称）
   - **Expiration**：建议 `90 days`（见下方说明）
   - **Scopes**：✅ 勾选 **`repo`**（完整勾选，包含所有子项）
   - **Scopes**：✅ 勾选 **`workflow`**
3. 点页面底部绿色按钮 **Generate token**
4. 复制生成的 token（格式 `ghp_xxxxxxxxxxxx`，**只显示一次**）
5. 到仓库 Settings → Secrets → `New repository secret`
   - Name：`REPO_TOKEN`
   - Value：粘贴刚才的 token

**⏰ Expiration 过期时间设置建议**：

| 过期时间 | 推荐度 | 适用场景 |
|---------|:------:|---------|
| 30 days | ⭐⭐ | 测试阶段，最安全但需频繁更新 |
| **90 days** | ⭐⭐⭐⭐⭐ | **日常使用，平衡安全与便利** |
| 1 year | ⭐⭐⭐ | 长期稳定运行，省心但风险略高 |
| No expiration | ⭐ | 不推荐，永不过期最不安全 |

**为什么推荐 90 天？**
- Godlike 续期是长期任务，token 太短频繁更新很烦
- 90 天足够安全，即使泄露最多影响 3 个月
- 配合 Cookie 回写，token 过期后 Cookie 还能用一段时间

**过期后如何更新？**
1. GitHub 会提前发邮件提醒
2. 打开 https://github.com/settings/tokens
3. 找到旧 token → 点 **Regenerate token**（不用删了重建）
4. 复制新 token
5. 到仓库 Settings → Secrets → 更新 `REPO_TOKEN` 的值

**⚠️ 注意**：
- Token 只显示一次，生成后立即复制保存
- 必须勾选 `repo` + `workflow` 权限，否则 Cookie 回写会失败
- 如果不需要 Cookie 缓存，可以不配，脚本会忽略回写错误

---

#### 📲 `TG_BOT_TOKEN`（Telegram Bot Token）

**用途**：通过 Telegram Bot 发送续期通知

**获取步骤**：
1. 在 Telegram 搜索 [@BotFather](https://t.me/BotFather)
2. 发送 `/newbot`
3. 按提示输入：
   - **Bot 名称**（显示名，如 `Godlike 续期通知`）
   - **Bot 用户名**（必须以 `bot` 结尾，如 `my_godlike_bot`）
4. 创建成功后，BotFather 会返回一个 token，格式如：
   ```
   1234567890:AAEhBP0xv-vXfXXXXXXXXXXXXXXXXXXXXXX
   ```
5. 到仓库 Settings → Secrets → `New repository secret`
   - Name：`TG_BOT_TOKEN`
   - Value：粘贴 token

---

#### 📲 `TG_CHAT_ID`（Telegram Chat ID）

**用途**：指定通知发送给哪个用户/群组

**获取步骤**：
1. 在 Telegram 搜索 [@userinfobot](https://t.me/userinfobot)
2. 给它发送任意消息（如 `/start`）
3. 它会回复你的 Chat ID，格式如：
   ```
   123456789
   ```
4. 到仓库 Settings → Secrets → `New repository secret`
   - Name：`TG_CHAT_ID`
   - Value：粘贴 Chat ID

**⚠️ 注意**：
- 必须先给你创建的 Bot 发送 `/start` 消息激活会话，否则 Bot 无法主动给你发消息
- 如果要发到群组，把 Bot 加入群组，Chat ID 用群组 ID（通常是负数，如 `-100123456789`）

---

#### 🌐 `PROXY_NODE`（代理节点，可选）

**用途**：如果 GitHub Actions IP 被 Godlike 封锁，配置代理切换 IP

**支持格式**（与 v2rayN/Clash 兼容的分享链接）：

| 协议 | 示例 |
|------|------|
| VLESS | `vless://uuid@host:port?type=ws&security=tls&sni=...` |
| VMess | `vmess://eyJhZGQiOi...` |
| Trojan | `trojan://password@host:port?type=ws&sni=...` |
| Shadowsocks | `ss://YWVzLTI1Ni1nY206...` |
| SOCKS5 | `socks5://user:pass@host:port` |

**获取步骤**：
1. 在你的代理客户端（v2rayN/Clash 等）找到节点
2. 右键 → **复制分享链接**（或 **导出为 URL**）
3. 到仓库 Settings → Secrets → `New repository secret`
   - Name：`PROXY_NODE`
   - Value：粘贴代理链接

**⚠️ 注意**：
- 不配置则直连访问
- 建议用住宅代理（机房 IP 可能被封锁）

---

### 3. 使用方法

#### 定时运行
默认每 3 小时执行一次（UTC），可在 `.github/workflows/Godlike_Renew.yml` 中修改 `cron`：
```yaml
on:
  schedule:
    - cron: '0 */3 * * *'  # 每 3 小时
```

#### 手动触发
1. 打开 `Actions` → `Godlike 续期`
2. 点 **Run workflow** → 选 `main` 分支 → 点绿色按钮

#### API 调用
```bash
curl -X POST \
  -H "Authorization: Bearer ghp_xxxx" \
  -H "Accept: application/vnd.github.v3+json" \
  https://api.github.com/repos/你的用户名/仓库名/actions/workflows/Godlike_Renew.yml/dispatches \
  -d '{"ref":"main"}'
```

---

## 📋 配置示例

### 最小配置（单账号 + 通知）

| Secret | Value |
|--------|-------|
| `GODLIKE_1` | `admin@example.com-----MyPassword123` |
| `REPO_TOKEN` | `ghp_xxxxxxxxxxxx` |
| `TG_BOT_TOKEN` | `1234567890:AAE...` |
| `TG_CHAT_ID` | `123456789` |

### 完整配置（多账号 + 代理）

| Secret | Value |
|--------|-------|
| `GODLIKE_1` | `admin1@example.com-----pwd1` |
| `GODLIKE_2` | `admin2@example.com-----pwd2` |
| `GODLIKE_3` | `admin3@example.com-----pwd3` |
| `REPO_TOKEN` | `ghp_xxxxxxxxxxxx` |
| `TG_BOT_TOKEN` | `1234567890:AAE...` |
| `TG_CHAT_ID` | `123456789` |
| `PROXY_NODE` | `vless://uuid@host:port?...` |

---

## 🐛 常见问题

**1. 登录失败**
- 检查 `GODLIKE_X` 格式（邮箱-----密码，5 个减号）
- 尝试在 Actions 的 Artifacts 中下载截图排查
- 如"Through login/password"按钮未出现，页面可能已更新，请提交 Issue

**2. Cookie 回写失败**
- 确保 `REPO_TOKEN` 已创建且具有 `repo` 和 `workflow` 权限
- 若不需要 Cookie 缓存，可忽略该警告

**3. "24小时上限"或"冷却期"是什么意思？**
- 24h 上限：服务器已攒满 24 小时运行时间，无法再增加，脚本视为成功并跳过
- 6 分钟冷却期：同一服务器续期后 6 分钟内无法再次续期，脚本会跳过，等待下次调度

**4. 截图在哪里查看？**
- 每次 Actions 运行结束，底部的 **Artifacts** 区域可下载截图包（仅在失败时上传）

**5. 没有收到 Telegram 通知？**
- 确认 `TG_BOT_TOKEN` 和 `TG_CHAT_ID` 正确
- 先在 Telegram 给你创建的 Bot 发送 `/start` 激活会话
- 群组通知：把 Bot 加入群组，且 Chat ID 用群组 ID（负数）

**6. 运行时间很长是否正常？**
- 正常，每个账号约 2-5 分钟，GitHub Actions 最长允许 6 小时

---

## 🛠️ 项目结构

| 文件 | 说明 |
|------|------|
| `main.py` | 主程序（Python + Playwright） |
| `.github/workflows/Godlike_Renew.yml` | GitHub Actions 工作流配置 |

---

## 🔒 安全建议

- 所有敏感信息均存储在 GitHub Secrets 中，Actions 日志已脱敏
- `REPO_TOKEN` 仅授予 `repo` 和 `workflow` 权限（Classic Token）
- 定期更新 Token 和密码

---

## 📄 许可证

MIT License

---

**⚠️ 免责声明**：本脚本仅供学习交流使用，使用者需遵守 Godlike 的服务条款。因使用本脚本造成的任何问题，作者不承担任何责任。
