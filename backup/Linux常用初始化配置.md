# Linux常用初始化配置

以下是您在新Linux系统上可能需要进行的一些初始化配置，涵盖了系统更新、用户管理、网络、安全和常用工具等方面。

#### 1. 系统更新与升级

这是第一步，确保您的系统软件包都是最新的，以获取最新的功能、修复和安全补丁。

* **Debian/Ubuntu 系列 (apt)**
  ```bash
  sudo apt update          # 更新软件包列表
  sudo apt upgrade -y      # 升级所有可升级的软件包
  sudo apt autoremove -y   # 清理不再需要的依赖包
  sudo apt clean           # 清理下载的包文件
  ```
* **CentOS/RHEL/Fedora 系列 (yum/dnf)**
  ```bash
  sudo dnf update -y       # 更新所有软件包 (CentOS 8+, Fedora)
  # 或者对于 CentOS 7 及以下
  sudo yum update -y       # 更新所有软件包
  ```

#### 2. 用户管理与权限

创建非root用户并配置sudo权限是最佳实践，以增强系统安全性。

* **创建新用户**
  ```bash
  sudo adduser your_username # 创建新用户并设置密码
  ```

  * 根据提示输入密码和其他信息。
* **赋予新用户sudo权限**
  * **方法一：添加到sudo组 (推荐)**

    ```bash
    sudo usermod -aG sudo your_username # Debian/Ubuntu
    sudo usermod -aG wheel your_username # CentOS/RHEL/Fedora
    ```
  * **方法二：编辑sudoers文件 (谨慎操作)**

    ```bash
    sudo visudo
    ```

    在文件中找到类似 `root ALL=(ALL:ALL) ALL` 的行，在其下方添加：
    ```
    your_username ALL=(ALL:ALL) ALL
    ```

    保存并退出（通常是 `Ctrl+X`, `Y`, `Enter` 或 `Esc`, `:wq`, `Enter`）。
* **切换到新用户**
  ```bash
  su - your_username
  ```

  后续操作尽量使用新用户，需要root权限时使用 `sudo`。

#### 3. SSH服务配置（远程访问）

如果您的服务器需要远程访问，SSH是必不可少的。

* **安装OpenSSH Server (如果未安装)**

  * **Debian/Ubuntu**
    ```bash
    sudo apt install openssh-server -y
    ```
  * **CentOS/RHEL/Fedora**
    ```bash
    sudo dnf install openssh-server -y
    sudo systemctl enable sshd --now # 启用并启动SSH服务
    ```
* **修改SSH默认端口 (可选，增强安全性)**

  ```bash
  sudo vi /etc/ssh/sshd_config
  ```

  找到 `Port 22`，取消注释并修改为其他端口号（例如 `Port 2222`）。
  ```
  #Port 22
  Port 2222
  ```
* **禁用root用户SSH登录 (强烈推荐)**
  在 `/etc/ssh/sshd_config` 中找到 `PermitRootLogin`，修改为：

  ```
  PermitRootLogin no
  ```
* **禁用密码登录，启用密钥登录 (强烈推荐)**

  * 在客户端生成SSH密钥对：`ssh-keygen`
  * 将公钥 (`~/.ssh/id_rsa.pub`) 复制到服务器的 `~/.ssh/authorized_keys` 文件中。
  * 在 `/etc/ssh/sshd_config` 中修改：
    ```
    PasswordAuthentication no
    PubkeyAuthentication yes
    ```
* **重启SSH服务**

  ```bash
  sudo systemctl restart sshd
  ```

  **注意：** 在禁用密码登录前，请务必确认您的SSH密钥登录已成功配置并测试通过，否则可能导致无法登录！

#### 4. 防火墙配置

为系统设置防火墙规则，限制不必要的端口访问。

* **Debian/Ubuntu (ufw)**
  ```bash
  sudo apt install ufw -y              # 安装ufw
  sudo ufw enable                      # 启用ufw
  sudo ufw allow 2222/tcp              # 允许SSH端口 (替换为您的SSH端口)
  sudo ufw allow http                  # 允许HTTP (80端口)
  sudo ufw allow https                 # 允许HTTPS (443端口)
  sudo ufw status verbose              # 查看防火墙状态
  ```
* **CentOS/RHEL/Fedora (firewalld)**
  ```bash
  sudo systemctl enable firewalld --now # 启用并启动firewalld
  sudo firewall-cmd --permanent --add-port=2222/tcp # 允许SSH端口
  sudo firewall-cmd --permanent --add-service=http  # 允许HTTP
  sudo firewall-cmd --permanent --add-service=https # 允许HTTPS
  sudo firewall-cmd --reload           # 重新加载防火墙规则
  sudo firewall-cmd --list-all         # 查看所有规则
  ```

#### 5. 时区和时间同步

确保系统时间正确，这对于日志记录、证书验证和各种服务都非常重要。

* **设置时区**
  ```bash
  sudo timedatectl set-timezone Asia/Shanghai # 设置为上海时区
  timedatectl status                          # 查看当前时区状态
  ```

  * 您可以使用 `timedatectl list-timezones` 查看所有可用时区。
* **启用NTP时间同步**
  * **Debian/Ubuntu**
    通常 `systemd-timesyncd` 已经默认启用。
    ```bash
    sudo apt install ntpdate -y # 如果需要手动同步
    sudo ntpdate -u pool.ntp.org # 手动同步一次
    ```
  * **CentOS/RHEL/Fedora**
    ```bash
    sudo dnf install chrony -y   # 安装chrony (推荐替代ntp)
    sudo systemctl enable chronyd --now # 启用并启动chronyd
    chronyc sources -v           # 查看NTP同步状态
    ```

#### 6. 安装常用工具

安装一些日常管理和故障排除的实用工具。

* **文本编辑器**
  ```bash
  sudo apt install vim -y # Debian/Ubuntu
  sudo dnf install vim -y # CentOS/RHEL/Fedora
  ```

  或者使用 `nano` (通常已预装)。
* **网络工具**
  ```bash
  sudo apt install net-tools iputils-ping traceroute curl wget -y # Debian/Ubuntu
  sudo dnf install net-tools iputils-ping traceroute curl wget -y # CentOS/RHEL/Fedora
  ```

  * `net-tools` 包含 `ifconfig`, `netstat` 等
  * `iputils-ping` 包含 `ping`
  * `traceroute` 用于路由追踪
  * `curl`, `wget` 用于命令行下载
* **系统监控工具**
  ```bash
  sudo apt install htop iotop iftop -y # Debian/Ubuntu
  sudo dnf install htop iotop iftop -y # CentOS/RHEL/Fedora
  ```

  * `htop` 交互式进程查看器
  * `iotop` 磁盘I/O监控
  * `iftop` 网络流量监控

#### 7. 内核参数优化 (可选，根据需求)

根据服务器用途，可能需要调整一些内核参数。

* **修改文件描述符限制 (例如，对于高并发服务)**

  ```bash
  sudo vi /etc/security/limits.conf
  ```

  添加：
  ```
  * soft nofile 65535
  * hard nofile 65535
  ```

  * `*` 表示所有用户，`nofile` 是文件描述符限制。
* **修改系统级文件描述符限制**

  ```bash
  sudo vi /etc/sysctl.conf
  ```

  添加或修改：
  ```
  fs.file-max = 65535
  ```

  应用更改：
  ```bash
  sudo sysctl -p
  ```

#### 8. 定期维护

* **设置自动更新 (可选，取决于环境)**
  * **Debian/Ubuntu (unattended-upgrades)**
    ```bash
    sudo apt install unattended-upgrades -y
    sudo dpkg-reconfigure --priority=low unattended-upgrades # 配置自动更新
    ```
  * **CentOS/RHEL/Fedora (dnf-automatic)**
    ```bash
    sudo dnf install dnf-automatic -y
    sudo vi /etc/dnf/automatic.conf # 配置更新模式
    sudo systemctl enable dnf-automatic.timer --now # 启用定时器
    ```
* **定期清理日志**
  通常由 `logrotate` 自动处理，但您可以检查其配置：
  ```bash
  ls /etc/logrotate.d/
  ```

---

完成这些初始化配置后，您的Linux系统将更加健壮、安全且易于管理。请根据您的具体需求和系统环境进行调整。