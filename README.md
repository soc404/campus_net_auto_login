# 校园网自动登录脚本

此仓库包含一个简单的 Python 脚本，用于在校园网环境中自动检测断网并尝试登录。

该脚本的设计目标：

- 在失去互联网访问（但仍能访问登录页面）时自动提交登录请求并恢复网络。 
- 提供可配置的检查间隔、重试策略和日志记录。 

## 快速开始

1. 克隆或下载该仓库到本地。
2. 编辑项目根目录下的 `config.json`，根据你学校的登录请求填写相关字段（下面有详细说明）。
3. 在终端运行：

```bash
python3 campus_auto_login.py
```

如果你希望脚本开机自启，可以参考下方的 systemd 示例将其作为服务运行。

## 配置说明（config.json）

示例配置文件会在首次运行时自动生成。重要字段说明：

- network.post_url: 提交登录请求的 URL（POST）。
- network.get_url: 用于检测“是否已登录成功”的页面 URL（通常为登录成功后的跳转页）。
- network.timeout: 网络请求超时时间（秒）。
- network.check_interval: 正常情况下的检查间隔（秒）。
- network.quick_check_interval: 发现异常后的快速重试间隔（秒）。
- network.max_quick_checks: 连续快速重试次数上限。
- network.internet_test_urls: 检测真实互联网连通性的外部 URL 列表。

- login.userId: 校园网账号（学号或工号）。
- login.password: 登录请求中使用的密码字段（可能需要加密或与校园网页面一致）。
- login.queryString: 从登录页面中得到的 queryString 参数（视学校认证页面而定）。
- login.passwordEncrypt: 是否对密码做了前端加密（字符串 "true"/"false"）。

- headers.Cookie: 浏览器中登录请求所带的 Cookie（如果登录请求需要）。
- headers.Referer: 请求来源（Referer），有些校园网会校验此字段。
- headers.User-Agent: 可选，默认示例中已包含常见 UA。

注意：不同学校的认证页面字段名称、加密方式、验证流程可能不同。请按浏览器开发者工具中观测到的请求结构填写 `config.json`。

## 如何获取需要的参数

1. 在浏览器中打开校园网登录页面，打开开发者工具（F12）。
2. 切换到 Network（网络）面板，找到提交登录的 POST 请求。
3. 在 Headers 面板中复制必要的请求头（Cookie、Referer、User-Agent 等）。
4. 在 Payload 或 Request Body 面板中复制表单字段（userId、password、queryString 等），并将其写入 `config.json` 的 `login` 部分。

如果登录页面对密码进行前端加密（例如 JS 加密），需要在 `config.json` 中标注并尽量复现加密逻辑，或直接使用浏览器中抓到的已加密密码字符串。

## 将脚本作为 systemd 服务（开机自启）

创建systemd服务文件
```bash
sudo nano /etc/systemd/system/campus-network.service
```

下面给出一个可用的 systemd 单元文件示例，替换其中的用户名与路径：

```ini
[Unit]
Description=Campus Network Auto Login
After=network.target

[Service]
Type=simple
User=your-username
WorkingDirectory=/home/your-username/campus_net_auto_login
ExecStart=/usr/bin/python3 /home/your-username/campus_net_auto_login/campus_auto_login.py
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
```

使用方法（在 Ubuntu/Debian 上）：

```bash
# 将上面的内容保存到 /etc/systemd/system/campus-network.service
sudo systemctl daemon-reload
sudo systemctl enable campus-network.service
sudo systemctl start campus-network.service
sudo systemctl status campus-network.service
# 查看实时日志
sudo journalctl -u campus-network.service -f
```

## 日志与故障排查

- 日志文件：`netconnect.log`。
- 如果脚本无法登录：请先手动在浏览器中完成登录并观察开发者工具中的请求，确认 `post_url`、字段名、Cookie 是否与 `config.json` 中一致。
- 如果登录成功页面不是 `<title>登录成功</title>`，可以修改脚本中检查登录状态的逻辑或将 `network.get_url` 改为一个稳定能反映已登录状态的页面。

常见问题：

- Q: 执行时提示配置文件不存在？
	A: 脚本会在首次运行时自动创建 `config.json` 示例文件，请根据注释修改后再次运行。

- Q: 登录请求返回 200 但无法访问外网？
	A: 可能校园网需要额外的计费/认证步骤或网络本身异常。检查 `netconnect.log` 中的详细请求/响应并根据需要调整字段或间隔。

