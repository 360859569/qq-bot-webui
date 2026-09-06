# QQ Bot WebUI

> 基于 FastAPI 与腾讯 QQ 开放平台官方接口（api-v2）的 QQ 机器人 Web 管理面板。
> 微信风格聊天界面 + macOS 式玻璃拟态 Dock，支持群聊/单聊实时收发、右键撤回、富媒体发送、
> 群管理、关键词自动回复、自定义菜单与指令面板可视化编辑。

---

## ✨ 功能特性

### 消息收发
- 🔌 **网关长连接**：官方 WSS 网关接入，自动鉴权（Identify）、心跳保活（op=1）、断线指数退避重连、消息去重
- 💬 群聊 / 单聊消息**实时接收与推送**（WebSocket 推送到网页，带未读角标）
- 📝 发送文本、Markdown、图片、视频、语音、文件（走官方**分片上传**完整流程：预上传 → 分片 PUT → 完成通知 → 合并，自动计算 md5/sha1，限并发、失败重试）
- ↩️ **右键撤回**消息（官方 2 分钟内可撤）、右键**回复**（被动回复 msg_id + msg_seq）、复制文本
- 😀 全套 Emoji + **自定义表情包**（上传后点击直接以图片消息发出）
- 🖼️ 图片 / 视频 / 语音 / 文件附件自动渲染（含语音转 WAV 播放链接）
- 📨 流式单聊消息、输入中状态（msg_type=6）、互动召回消息、内嵌键盘、引用回复等官方消息能力均已封装

### 群与用户
- 👥 群真实**名称、头像、成员数**获取（自动缓存，聊天列表直接显示）
- 🔍 私聊昵称：官方无单聊用户信息接口，本项目通过**跨场景昵称缓存**（群消息/加群申请中的 member_openid ↔ 昵称）自动关联出私聊真实昵称，支持手动备注
- 🛠️ **群管理面板**：群信息、成员列表（游标分页、踢人）、加群审批（同意/拒绝/拒绝并拉黑）、黑名单、机器人状态、会话限制设置、加群审批策略、指令面板、自定义菜单、通用接口调试

### 自动化
- ⚡ **关键词自动回复**：模糊（包含）/ 精准（完全匹配）两种模式，可选仅群聊 / 仅单聊 / 全部范围，每条规则独立开关，持久化存储，命中自动回复（防机器人互刷）

### 界面
- 🍎 macOS 风格 **Dock 导航栏**：图标鼠标靠近放大、悬浮标签、激活指示点；手机端自动隐藏，右下角悬浮按钮呼出
- 🪟 **动态玻璃拟态**（Liquid Glass 风格）：高饱和毛玻璃 + 流动光泽动画，适配系统"减弱动态效果"偏好
- 📱 手机 / 电脑自适应
- 🖥️ 实时运行日志面板（服务端日志 WebSocket 推送）
- 🤖 **机器人监控弹窗**：点击 Dock 里的机器人头像弹出（点击任意空白处关闭）——展示机器人 AppID、QQ 号、头像、程序名称与介绍、Logo 与"禁止违法违规使用"标签，并内置**服务器实时监控**（CPU / 内存 / 磁盘 / 宽带上下行速率 / 连接数，每 2 秒刷新）
- 🔐 安全：登录验证码（服务端 SVG 渲染）、Bearer 会话鉴权（30 天，重启不掉线）、登录限流、上传白名单与随机文件名、XSS 防护、`/img` 目录访问保护

### 官方接口覆盖
基于官方文档提取的完整规范见 [`docs/`](docs/)（`spec_events.md` 事件与网关 / `spec_messages.md` 消息类 / `spec_groups.md` 群管与频道），实现模块：
- `qq_gateway.py`：网关长连接（鉴权、心跳、事件分发、重连）
- `qq_messages_api.py`：发送 / 撤回 / 富媒体分片上传 / 流式 / 互动回应 / 分享链接
- `qq_group_api.py`：群管理、用户、频道、菜单、指令面板共 33 个接口

---

## 📁 目录结构

```
qq/
├── main.py               # FastAPI 主程序（路由 / 鉴权 / 事件处理 / 落盘）
├── qq_http.py            # QQ 开放平台 HTTP 共享模块
├── qq_gateway.py         # 网关 WSS 长连接
├── qq_messages_api.py    # 消息类 API 封装
├── qq_group_api.py       # 群管/用户/频道/菜单 API 封装
├── static/
│   ├── index.html        # 单文件前端
│   ├── favicon.ico       # 网站图标（可替换）
│   └── logo.png          # 登录页 Logo（可替换）
├── docs/                 # 官方文档提取的接口规范（spec_*.md）
├── 1.json                # 默认配置模板（复制为 config.json 使用）
├── nginx.conf            # 域名访问反向代理配置（含 WebSocket 支持）
├── requirements.txt
└── README.md
```

运行时自动生成：`data/group/`、`data/private/`（聊天记录，JSONL 格式）、`data/keywords.json`（关键词规则）、`data/nicknames.json`（昵称缓存）、`data/sessions.json`（登录会话）、`img/`（表情与媒体文件）。

---

## 🌍 环境要求

| 项 | 要求 |
|---|---|
| 系统 | Windows / Linux / macOS，公网或可访问到 QQ 开放平台的网络环境 |
| Python | **3.9 及以上**（开发验证环境：3.11.5） |
| 依赖 | 见 `requirements.txt`：fastapi / uvicorn / websockets / aiohttp / python-multipart / **psutil**（服务器监控） |
| 账号 | [QQ 开放平台](https://q.qq.com/qqbot/) 注册并创建机器人应用（获取 AppID / AppSecret），在管理端**订阅群聊与单聊事件（1<<25）**，并把机器人拉进群 |
| 端口 | 默认 `5200`（可配置），需防火墙放行或反向代理 |

---

## 🚀 快速开始

```bash
# 1. 克隆项目
git clone https://github.com/360859569/qq-bot-webui.git
cd qq-bot-webui/qq

# 2. 安装依赖（建议使用虚拟环境）
python -m venv venv
# Windows:
venv\Scripts\activate
# Linux / macOS:
source venv/bin/activate

pip install -r requirements.txt

# 3. 配置：把模板 1.json 复制为 config.json，然后填写你的机器人信息
# Windows:
copy 1.json config.json
# Linux / macOS:
cp 1.json config.json

# 4. 启动
python main.py

# 5. 浏览器访问
# http://127.0.0.1:5200
# 用 config.json 里的 web_user / web_pass 登录
```

> ⚠️ **重要**：仓库里的 `1.json` 是**默认配置模板**（避免把真实密钥提交到 GitHub）。
> 首次使用时请把它**复制并重命名为 `config.json`**，再填入你自己的 AppID / AppSecret / 登录密码。
> 建议把 `config.json` 加入 `.gitignore`，请勿提交真实密钥。

---

## ⚙️ 配置说明（config.json）

| 键 | 说明 | 环境变量覆盖 |
|---|---|---|
| `bot_appid` | 机器人 AppID（QQ 开放平台获取） | `QQ_BOT_APPID` |
| `bot_secret` | 机器人 AppSecret | `QQ_BOT_SECRET` |
| `is_sandbox` | 沙箱模式。`true` 时使用沙箱域名（仅测试环境机器人用），**正式机器人务必为 `false`** | - |
| `api_base` | API 域名，默认 `https://api.sgroup.qq.com` | `QQ_API_BASE` |
| `web_port` | 面板端口，默认 `5200` | `WEB_PORT` |
| `web_user` | 面板登录用户名 | `WEB_USER` |
| `web_pass` | 面板登录密码 | `WEB_PASS` |
| `intents` | 网关基础订阅位掩码，默认 `33554432`（=1<<25，群聊+单聊事件） | - |
| `extra_intents` | 附加订阅位列表：`24`=群成员事件，`26`=互动事件；若某位未在平台订阅，网关会自动降级并记录日志 | - |

> 敏感信息（AppID/Secret/登录账密）建议用**环境变量**覆盖，避免明文落盘。

---

## 🖥️ 部署教程

### 方式一：Windows 直接运行

```powershell
cd qq
pip install -r requirements.txt
python main.py
```

**开机自启（任务计划程序）**：
1. `Win + R` → `taskschd.msc` 打开任务计划程序 → 创建任务
2. 触发器：`启动时`；操作：`启动程序` → 程序填 `python.exe` 完整路径，参数填 `main.py` 完整路径，起始于填项目目录
3. 勾选"使用最高权限运行"与"不管用户是否登录都要运行"

### 方式二：Linux systemd 守护

```bash
# 安装依赖
cd /opt/qq-bot-webui/qq
python3 -m venv venv && source venv/bin/activate
pip install -r requirements.txt
cp 1.json config.json   # 填写配置

# 创建服务
sudo nano /etc/systemd/system/qqbot.service
```

```ini
[Unit]
Description=QQ Bot WebUI
After=network.target

[Service]
Type=simple
WorkingDirectory=/opt/qq-bot-webui/qq
ExecStart=/opt/qq-bot-webui/qq/venv/bin/python main.py
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now qqbot
sudo systemctl status qqbot      # 查看状态
```

### 方式三：宝塔面板部署（推荐）

1. **上传代码**：宝塔文件管理 → 进入 `/www/wwwroot/` → 新建目录 `qqbot` → 上传项目文件（或 `git clone`）；上传后进入目录，把 `1.json` **复制重命名为 `config.json`** 并填写机器人信息
2. **安装 Python 项目管理器**：宝塔「软件商店」搜索安装 **Python项目管理器 2.x**（依赖 python 环境，宝塔会自动安装，选择 Python 3.10+）
3. **添加项目**：
   - 打开 Python 项目管理器 →「添加项目」
   - 项目名称：`qqbot`
   - 运行文件：`/www/wwwroot/qqbot/qq/main.py`
   - 运行目录：`/www/wwwroot/qqbot/qq`
   - Python 版本：选择已安装的 3.10+
   - 启动方式：`python`（管理器会用所选解释器运行 main.py）
   - 端口：`5200`；勾选「开机启动」
   - 提交后管理器自动创建虚拟环境、`pip install -r requirements.txt` 并启动
4. **放行端口**：宝塔「安全」→ 放行 `5200` 端口（若只走反向代理可只放行内网，不对外）
5. **域名访问（可选，推荐）**：想用域名访问请参考下文「**🔗 域名访问与反向代理**」章节——注意务必转发 WebSocket 升级头，否则实时消息不推送
6. **日志与维护**：Python 项目管理器内可直接查看运行日志、重启、停止项目

> 宝塔部署后若"收不到消息"：确认 `is_sandbox=false`，并在 QQ 开放平台管理端确认机器人**已订阅群聊/单聊事件**、已加入目标群，群里开启了「接收所有消息」（或有人 @机器人）。

---

## 🔗 域名访问与反向代理（伪静态 / Nginx 配置）

项目内置了开箱即用的反代配置：[`nginx.conf`](nginx.conf)。核心要点：

- ⭐ **必须转发 WebSocket 升级头**（`Upgrade` / `Connection: upgrade`）——聊天消息与日志的实时推送走 WebSocket，缺少这两行会导致页面能打开但**收不到实时消息**
- 上传大小限制（面板富媒体最大 200MB）
- 长连接超时放宽（WebSocket 空闲保持）

### Nginx 配置内容（伪静态）

```nginx
server {
    listen 80;
    server_name bot.example.com;          # ← 改成你的域名

    # 上传大小限制（面板富媒体上传最大 200MB）
    client_max_body_size 300m;

    location / {
        proxy_pass http://127.0.0.1:5200;  # ← 改成面板实际端口

        proxy_http_version 1.1;            # WebSocket 必须 HTTP/1.1

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        # ★ WebSocket 升级头（实时消息推送必需）
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";

        proxy_read_timeout 300s;
        proxy_send_timeout 300s;
    }
}
```

### 使用教程（宝塔面板）

1. **解析域名**：到域名服务商处添加 A 记录，把 `bot.example.com` 指向服务器 IP
2. **添加站点**：宝塔「网站」→「添加站点」→ 域名填 `bot.example.com`，PHP 版本选**纯静态**（无需 PHP）
3. **写入反代配置（推荐方式：直接改站点配置文件）**：
   - 站点列表 → 你的站点 →「设置」→「配置文件」
   - 把 `server {}` 里原来的 `location / { ... }` 整段**替换**为上面配置中的 `location / { ... }`（保留宝塔自动生成的其他部分不动）
   - 同时把最上方的 `client_max_body_size 300m;` 加到 `server {}` 块内
   - 保存（宝塔会自动 `nginx -t` 校验并重载；校验失败说明粘贴位置不对，按提示修正）
4. **（可选）HTTPS**：站点设置 →「SSL」→ 申请免费的 Let's Encrypt 证书 → 开启「强制 HTTPS」；启用后前端自动切换 `wss://`，无需改任何代码
5. **验证**：浏览器访问 `https://bot.example.com` → 登录 → 打开「实时日志」页能看到日志滚动、群里发消息面板能实时刷新，即代表 WebSocket 反代成功

> ⚠️ 宝塔的「反向代理」图形化功能**默认不带 WebSocket 升级头**，用它添加反代后实时消息会失效；如果已经用图形化功能添加过反代，请再到「配置文件」里手动补上 `Upgrade` / `Connection` 两行。
>
> 命令行部署则直接：`cp nginx.conf /etc/nginx/conf.d/qqbot.conf` → `nginx -t && nginx -s reload`

---

## ❓ 常见问题（FAQ）

**Q：登录后收不到群消息？**
A：依次检查 ① `config.json` 的 `is_sandbox` 是否为 `false`（沙箱机器人连生产群收不到消息）；② 开放平台管理端是否订阅了群聊与单聊事件（1<<25）；③ 机器人是否在目标群里；④ 若只想收 @机器人 的消息，需要群里开启对应设置，否则发送消息时会收到 `GROUP_AT_MESSAGE_CREATE` 事件；⑤ 打开面板「实时日志」页查看网关状态（READY 表示连接成功）。

**Q：成员列表 / 踢人接口报 `11253 应用无接口访问权限`？**
A：这些接口仅对白名单机器人开放，需要在 QQ 开放平台向运营申请对应权限；群信息、消息收发不受影响。

**Q：撤回消息失败？**
A：官方限制：消息发送超过 **2 分钟**不可撤回；机器人是普通成员时只能撤回自己发的消息，群管理员可撤回他人消息。

**Q：发送语音（mp3）失败？**
A：官方语音消息仅支持 **silk** 格式。本项目会自动把不支持的音频降级为**文件类型**发送，对方会收到一个文件。

**Q：私聊不显示对方昵称？**
A：官方没有单聊用户信息接口且单聊事件不带昵称。本项目会自动关联该用户在群里出现过的昵称；如果从未在群里出现，可在私聊页顶部点「**备注昵称**」手动设置。

**Q：刷新页面 / 重启服务需要重新登录吗？**
A：不需要。登录会话保存在服务端 `data/sessions.json`（有效期 30 天），服务重启后依然有效；只有会话过期或点击退出登录才需要重新登录。

**Q：换了 favicon / 登录页 Logo 怎么改？**
A：直接替换 `static/favicon.ico` 和 `static/logo.png`（保持文件名），刷新页面即可生效。

**Q：机器人监控弹窗里服务器监控显示 N/A？**
A：服务器监控依赖 `psutil`，请先执行 `pip install -r requirements.txt`（或 `pip install psutil`）并重启服务；个别虚拟主机环境可能禁止读取系统指标，属正常限制。

**Q：端口被占用？**
A：修改 `config.json` 的 `web_port`，或设置环境变量 `WEB_PORT` 后重启。

---

## 🗃️ 数据与安全说明

- 聊天记录以 JSONL 明文存放在 `data/group/`、`data/private/`（按群/用户 OpenID 分文件）
- 登录密码、AppSecret 仅用于服务端调用与登录校验，建议通过环境变量注入
- 本项目仅供学习与个人/授权场景使用，请遵守 [QQ 开放平台服务协议](https://q.qq.com/qqbot/) 与相关法律法规

## 📄 License

[MIT](LICENSE) © 2026

## 🙏 参考

- [腾讯 QQ 机器人官方文档（api-v2）](https://bot.q.qq.com/wiki/develop/api-v2/)
- 接口规范提取文档：[`docs/spec_events.md`](docs/spec_events.md) / [`docs/spec_messages.md`](docs/spec_messages.md) / [`docs/spec_groups.md`](docs/spec_groups.md)
