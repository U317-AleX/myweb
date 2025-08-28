# 📚 Scientific Internet
---
## 关于22端口被机场屏蔽问题

### 为什么要使用443端口
- SSH 协议和端口的默认规则

`GitHub` 的 `SSH` 方式远程连接（`git@github.com:...`）底层就是使用 `SSH` 协议。

`SSH` 协议的默认端口就是 `22`，所以不需要在 `URL` 里额外写。

比如 `ssh -T git@github.com` 实际就是连 `github.com:22`。

如果不是 `22`，就需要显式写在配置里（比如改成 `443`）。

- 为什么 `Git` 默认用 `22`

因为 `SSH` 最初就是为了远程安全登录服务器而设计的，大家约定俗成地都用 `22` 端口。
`Git` 只是借助了 `SSH` 来做认证和加密传输，所以 自然继承了 `SSH` 的默认端口 `22`。

- 为什么 `GitHub` 还支持 `443` 端口

在一些环境（比如公司内网、防火墙、学校网络）里，`22` 端口常常被封掉（因为默认就是 `SSH`，被认为风险大）。

但 `443` 端口（`HTTPS` 用的）几乎不会被封，因为要上网。

所以 `GitHub` 提供了备用的 `ssh.github.com:443`，配合 `.ssh/config` 就能用。

这样就能在端口受限的网络下继续使用 `SSH`，而不用退回到 `HTTPS + token` 的方式。

### 🚀 在 Windows + PowerShell 下配置GitHub SSH 走 443 端口流程总结

#### 为什么不用修改远程仓库的 `URL`？

你远程仓库的 `URL` 是：

`git@github.com:user_name/repo_name.git`


这是 标准的 `GitHub SSH` 地址，是官方推荐的写法。

`SSH` 的配置 （`~/.ssh/config`） 可以让你在 不改 `URL` 的情况下，把 `github.com` 重定向到 `ssh.github.com:443`。

所以，只要 `config` 生效了，`Git` 在用 `git@github.com` 连接时，实际上会走 `ssh.github.com:443`，你不需要去 `git remote set-url`。

#### Windows 上 `.ssh/config` 的格式要求

文件路径：`C:\Users\user_name\.ssh\config`

注意事项：

文件必须是 纯文本，不能有 `UTF-8 BOM`。

`BOM` 就是报错里的 `\357\273\277`，会导致 `Bad configuration option`。

换行符要用 `LF` 或 `CRLF` 都行，`OpenSSH` 在 `Windows` 上能识别，但 `BOM` 不行。

缩进建议用 空格，不要用 `Tab`（虽然理论上能解析，但不保险）。

#### 正确的配置内容
`Host github.com`
`    HostName ssh.github.com`
`    User git`
`    Port 443`


解释：

`Host github.com`：匹配远程 `URL` 里的 `github.com`

`HostName ssh.github.com`：实际连接到 `ssh.github.com`

`User git`：`GitHub SSH` 的固定用户

`Port 443`：指定用 `443` 端口绕过防火墙

#### 在 PowerShell 里创建配置文件

推荐用 ascii 或 utf8NoBOM，避免带 BOM。

```powershell
@"
Host github.com
    HostName ssh.github.com
    User git
    Port 443
"@ | Out-File -FilePath $env:USERPROFILE\.ssh\config -Encoding ascii -Force
```

这样写入的文件就是干净的，无 `BOM`。

#### 验证配置是否生效

运行：

```bash
ssh -T -v git@github.com
```

如果成功，会看到：

`debug1: Reading configuration data C:\\Users\\user_name\\.ssh\\config`
`debug1: Hostname ssh.github.com port 443 user git`

并提示：

`Hi user_name! You've successfully authenticated...`

####  Git 测试

<!-- end list -->

```bash
git fetch origin
```

```bash
git push origin main
```

如果不再报 `443` 或 `connection refused`，说明配置完成。

✅ 这样做的好处是：

不需要改远程仓库 `URL`，配置更优雅。

Windows 的 BOM 问题被解决。

`.ssh/config` 生效后，所有仓库都会自动走 `443`，不用每次单独设置。

---

## 内网穿透

以下是使用 Ngrok 在只有 Linux CLI 的环境下进行内网穿透的完整步骤：

###  下载 Ngrok 压缩包

首先，你需要使用 `curl` 或 `wget` 命令下载 Ngrok 的 Linux 版本压缩包。

你可以直接访问 Ngrok 官网的下载链接。通常，它的格式为：`https://bin.equinox.io/c/4VmDzA7iaHb/ngrok-stable-linux-amd64.zip`。

```bash
wget [https://bin.equinox.io/c/4VmDzA7iaHb/ngrok-stable-linux-amd64.zip](https://bin.equinox.io/c/4VmDzA7iaHb/ngrok-stable-linux-amd64.zip)
```

或者

```bash
curl -O [https://bin.equinox.io/c/4VmDzA7iaHb/ngrok-stable-linux-amd64.zip](https://bin.equinox.io/c/4VmDzA7iaHb/ngrok-stable-linux-amd64.zip)
```

-  解压 Ngrok
    下载完成后，你会得到一个 `.zip` 文件。使用 `unzip` 命令来解压它。如果你的系统没有 `unzip`，需要先安装。

<!-- end list -->

```bash
sudo apt-get install unzip  # Debian/Ubuntu 系统
sudo yum install unzip      # CentOS/RHEL 系统
```

然后执行解压命令：

```bash
unzip ngrok-stable-linux-amd64.zip
```

解压后，你会得到一个名为 `ngrok` 的可执行文件。

###  连接你的 Ngrok 账号

为了使用 Ngrok，你需要连接你的账号。这个步骤会使用你的认证令牌（Authtoken）。

获取认证令牌：登录 Ngrok 官网（`ngrok.com`），在你的仪表盘上找到 “`Your Authtoken`” 页面。复制那里的认证令牌。

配置令牌：在你的 Linux CLI 中，运行以下命令，并把 `<你的认证令牌>` 替换成你刚刚复制的令牌。

```bash
./ngrok authtoken <你的认证令牌>
```

这个命令会在你的用户主目录下创建一个名为 `.ngrok2/ngrok.yml` 的配置文件，并保存你的认证信息。你只需运行一次即可。

### 启动内网穿透服务

假设你的本地服务正在 VPS 的 `8080` 端口上运行，你可以用以下命令启动 Ngrok，将这个端口暴露到公网。

```bash
./ngrok http 8080
```

运行后，Ngrok 会在终端界面显示连接状态和为你生成的公网 URL。你可以在这个终端界面看到类似下面这样的输出：

`ngrok                                                                                                                                                                                                                                               (Ctrl+C to quit)`

`Session Status              online`
`Account                     YourName (Plan: Free)`
`Version                     3.4.0`
`Region                      United States (us)`
`Web Interface               http://127.0.0.1:4040`
`Forwarding                  https://a1b2c3d4.ngrok.io -> http://localhost:8080`

`Connections                 ttl     opn     rt1     p50     p90`
`                              0       0       0.00    0.00    0.00 `
`Forwarding` 行中的 `https://...ngrok.io` 就是你的服务在公网上的地址。现在，你就可以在任何地方通过这个网址访问你的服务了。

###  在后台运行 Ngrok（可选）

如果你想让 Ngrok 在终端关闭后继续运行，可以利用 `nohup` 命令或者 `screen/tmux` 等工具。

使用 `nohup`：

```bash
nohup ./ngrok http 8080 > ngrok.log 2>&1 &
```

`nohup`：让命令在后台运行，忽略挂断信号。

`> ngrok.log 2>&1`：将所有输出（包括标准输出和错误输出）重定向到 `ngrok.log` 文件。

`&`：将命令放入后台执行。

运行这个命令后，你可以通过 `cat ngrok.log` 来查看 Ngrok 提供的公网 URL。