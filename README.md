# 校园网自动登录脚本

## 配置说明

### 1. 配置文件 (config.json)
将 `config.json` 文件中的以下字段替换为实际值：

- `network.get_url`: 登录成功页面的URL
- `login.userId`: 校园网账号
- `login.password`: 加密后的密码
- `login.queryString`: 从登录页面获取的queryString参数
- `headers.Cookie`: 浏览器中的Cookie值
- `headers.Referer`: Referer URL

### 2. 获取配置参数的方法
打开浏览器开发者工具 (F12)

访问校园网登录页面

在Network标签页找到登录请求

复制所需的Headers和Payload参数

设置post的请求头，浏览器点击F12，在Netword中选中post请求，点击Headers、request header面板中查看

设置post的请求数据，浏览器点击F12，在Netword中选中post请求，点击payload面板中查看

### 3.运行方式

```python
python campus_auto_login.py
```

### 4.在Ubuntu上让脚本常驻运行

1. 创建systemd服务文件

```bash
sudo nano /etc/systemd/system/campus-network.service
```

2. 添加以下内容

```ini
[Unit]
Description=Campus Network Auto Login
After=network.target

[Service]
Type=simple
User=你的用户名
WorkingDirectory=/home/你的用户名/脚本所在路径
ExecStart=/usr/bin/python3 /home/你的用户名/脚本所在路径/你的脚本名.py
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
```

3. 启用并启动服务

```bash
# 重新加载systemd配置
sudo systemctl daemon-reload

# 启用服务（开机自启）
sudo systemctl enable campus-network.service

# 启动服务
sudo systemctl start campus-network.service

# 检查服务状态
sudo systemctl status campus-network.service

# 查看日志
sudo journalctl -u campus-network.service -f
```