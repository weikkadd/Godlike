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

在仓库 `Settings` → `Secrets and variables` → `Actions` 中添加：

| Secret 名称 | 必填 | 说明 | 示例 |
|------------|:---:|------|------|
| `GODLIKE_1` | ✅ | 主账号，格式 `邮箱-----密码` | `admin@example.com-----MyPassword` |
| `GODLIKE_2` ~ `GODLIKE_5` | ❌ | 额外账号（最多到5） | 同上 |
| `REPO_TOKEN` | ✅ | 用于自动回写 Cookie 的 GitHub PAT | `ghp_xxxxxxxxxxxx` |
| `TG_BOT_TOKEN` | ❌ | Telegram Bot Token | `1234567890:AAE...` |
| `TG_CHAT_ID` | ❌ | 接收通知的 Chat ID | `123456789` |
| `PROXY_NODE` | ❌ | 代理节点链接（可选） | 见代理格式说明 |

账号 Secret 首次只需提供邮箱和密码，脚本会在登录成功后自动将 Cookie 追加到 Secret 末尾，后续优先使用 Cookie。

### 2. 代理节点格式（可选）

| 协议 | 示例 |
|------|------|
| VLESS | `vless://uuid@host:port?type=ws&security=tls&sni=...` |
| VMess | `vmess://eyJhZGQiOi...` |
| Trojan | `trojan://password@host:port?type=ws&sni=...` |
| Shadowsocks | `ss://YWVzLTI1Ni1nY206...` |
| SOCKS5 | `socks5://user:pass@host:port` |

### 3. 使用方法

- **定时运行**：默认每 3 小时执行一次（UTC），可在 `.github/workflows/Godlike_Renew.yml` 中修改 `cron`。
- **手动触发**：`Actions` → `Godlike 续期` → `Run workflow`。
- **API 调用**：
  ```bash
  curl -X POST \
    -H "Authorization: Bearer ghp_xxxx" \
    -H "Accept: application/vnd.github.v3+json" \
    https://api.github.com/repos/你的用户名/仓库名/actions/workflows/Godlike_Renew.yml/dispatches \
    -d '{"ref":"main"}'
  ```

---

## 📋 Telegram 通知配置

- `TG_BOT_TOKEN`：向 [@BotFather](https://t.me/BotFather) 发送 `/newbot` 创建机器人获取。
- `TG_CHAT_ID`：向 [@userinfobot](https://t.me/userinfobot) 发送任意消息获取。

配置后，续期成功、24h 上限、冷却期、失败都会推送带截图的图文消息。

---

## 🐛 常见问题

**1. 登录失败**
- 检查 `GODLIKE_X` 格式（邮箱-----密码）。
- 尝试在 Actions 的 Artifacts 中下载截图排查。
- 如"Through login/password"按钮未出现，页面可能已更新，请提交 Issue。

**2. Cookie 回写失败**
- 确保 `REPO_TOKEN` 已创建且具有 `repo` 和 `workflow` 权限。
- 若不需要 Cookie 缓存，可忽略该警告。

**3. "24小时上限"或"冷却期"是什么意思？**
- 24h 上限：服务器已攒满 24 小时运行时间，无法再增加，脚本视为成功并跳过。
- 6 分钟冷却期：同一服务器续期后 6 分钟内无法再次续期，脚本会跳过，等待下次调度。

**4. 截图在哪里查看？**
- 每次 Actions 运行结束，底部的 **Artifacts** 区域可下载截图包（仅在失败时上传）。

**5. 没有收到 Telegram 通知？**
- 确认 `TG_BOT_TOKEN` 和 `TG_CHAT_ID` 正确。
- 先在 Telegram 给 Bot 发送 `/start` 激活会话。

**6. 运行时间很长是否正常？**
- 正常，每个账号约 2-5 分钟，GitHub Actions 最长允许 6 小时。

---

## 🛠️ 项目结构

| 文件 | 说明 |
|------|------|
| `main.py` | 主程序（Python + Playwright） |
| `.github/workflows/Godlike_Renew.yml` | GitHub Actions 工作流配置 |

---

## 🔒 安全建议

- 所有敏感信息均存储在 GitHub Secrets 中，Actions 日志已脱敏。
- `REPO_TOKEN` 仅授予 `repo` 和 `workflow` 权限（Classic Token）。
- 定期更新仓库以获取最新修复。

---

## 📄 许可证

MIT License

---

**⚠️ 免责声明**：本脚本仅供学习交流使用，使用者需遵守 Godlike 的服务条款。因使用本脚本造成的任何问题，作者不承担任何责任。
