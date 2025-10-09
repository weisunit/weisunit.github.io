# Proxmox VE 高可用集群搭建指南

## 1. 介绍

[Proxmox VE (PVE)](https://www.proxmox.com/en/downloads) 是一款开源的虚拟化平台，基于 KVM (Kernel-based Virtual Machine) 和 LXC (Linux Containers) 技术，支持虚拟机和容器的运行。PVE 还提供高可用集群管理、软件定义存储、备份和恢复以及网络管理等企业级功能。

## 2. 测试环境规划

### 2.1 节点数量

- 官方推荐至少 3 节点搭建高可用集群
- 3 台虚拟机用于测试，可后续添加新节点
- 2 节点也可搭建集群部署 Ceph，但存在风险：**如果有节点离线，Ceph 直接不可用**
- 实际使用中保证至少3节点,最多32个节点

### 2.2 硬盘分配

- 50G 盘用于安装操作系统（实际环境中应使用两块 SSD 做 RAID1）
- 512G 盘作为数据存储（可按需求添加）
- 使用 Ceph 分布式存储时，存储盘不要做 RAID，直通到系统由 Ceph 统一管理

### 2.3 网络规划

```
管理网络：192.168.1.0/24
集群网络：192.168.10.0/24
Ceph存储网络：192.168.20.0/24
```

- 根据实际需求和网卡数量自行规划IP
- 集群和存储网络尽可能使用 10Gbps 及以上速率的网卡

## 3. 系统安装与配置

### 3.1 安装步骤

![QQ20250427-232533.png](https://cdn.jsdelivr.net/gh/weisunit/note-gen-image-sync@main/b1c22467-82fe-4454-9793-281be40e77cd.png)

1. [下载 ISO 镜像](https://www.proxmox.com/en/downloads)
2. 安装步骤：选择磁盘，关闭 SWAP，设置 FQDN 和固定 IP
3. 安装完成后使用 `https://ip地址:8006` 登录管理界面

### 3.2 修改网络配置

1. 在`系统->网络`菜单创建 `Linux Bridge`

   ![QQ20250428-001017.png](https://cdn.jsdelivr.net/gh/weisunit/note-gen-image-sync@main/8eb72b88-298d-4277-9f9c-579fbae8dcb7.png)
2. 修改 hosts 文件：

```xml
# 管理
192.168.1.220 pve1.weisunit.com 
192.168.1.230 pve2.weisunit.com 
192.168.1.240 pve3.weisunit.com 
# 集群
192.168.10.220 pve1.weisunit.com 
192.168.10.230 pve2.weisunit.com 
192.168.10.240 pve3.weisunit.com 
# 存储
192.168.20.220 pve1.weisunit.com 
192.168.20.230 pve2.weisunit.com
192.168.20.240 pve3.weisunit.com 
```

- 所有节点hosts都要修改
- 建议自建DNS服务器，每个节点DNS地址改为自建DNS地址

## 4. 配置集群

![QQ20250428-002002.png](https://cdn.jsdelivr.net/gh/weisunit/note-gen-image-sync@main/f0c10281-0a92-4f4e-9760-5dec6c476712.png)

1. 在`数据中心->集群`中点击`创建集群`
2. 点击`加入信息`复制信息
3. 在其他节点`数据中心->集群`中点击`加入集群`
4. 粘贴信息并填入密码，选择正确的集群网络

   ![QQ20250428-002320.png](https://cdn.jsdelivr.net/gh/weisunit/note-gen-image-sync@main/e1122537-4f72-4dca-a3f9-286b270a1db5.png)

集群特性：

- 任意节点网页可查看所有节点
- 各节点配置ssh密钥互相免密码登录
- `/etc/pve`配置文件自动同步
- 节点要求时间同步（可使用`ntpdate ntp.ntsc.ac.cn`）

## 5. Ceph安装配置

### 5.1 安装步骤

1. 在`数据中心->ceph`安装ceph（每个节点都要安装）

   ![QQ20250428-003635.png](https://cdn.jsdelivr.net/gh/weisunit/note-gen-image-sync@main/6d535768-883d-45b3-a1df-f1199305da6b.png)
2. 安装完成后在`数据中心->Ceph`中点击`配置 Ceph`
3. 在节点`Ceph->监视器`中为其他节点添加`Monitor`和`Manager`

### 5.2 OSD配置

1. 在节点`Ceph->OSD`菜单中为每个磁盘创建`OSD`
2. OSD配置完成后在节点`Ceph->资源池`中配置资源池

   ![QQ20250428-005111.png](https://cdn.jsdelivr.net/gh/weisunit/note-gen-image-sync@main/de84ec7f-85a9-4691-a804-d1841a6fadca.png)

### 5.3 CephFS配置

- 存储只能用于存储虚拟机磁盘文件
- 如需存储ISO镜像或备份文件需创建CephFS
- 在节点`Ceph->CephFS->元数据服务器`为每台主机创建
- 然后创建CephFS

## 6. 启用HA

![QQ20250428-010158.png](https://cdn.jsdelivr.net/gh/weisunit/note-gen-image-sync@main/d0444a8b-481f-48a4-be5c-d18f28d546d3.png)

1. 创建虚拟机并将磁盘放到资源池上（如不在需先迁移）
2. 在`数据中心->HA->资源`添加虚拟机
3. 等待状态变成`started`后HA启用
4. 虚拟机所在节点异常时会在另一节点重启（120秒内完成故障转移）

## 7. 国内源配置

```xml
# 修改 Debian 源
sed -i 's|http://ftp.debian.org|https://mirrors.ustc.edu.cn|g' /etc/apt/sources.list
sed -i 's|http://security.debian.org|https://mirrors.ustc.edu.cn/debian-security|g' /etc/apt/sources.list
# 修改 PVE 源
sed -i 's|enterprise.proxmox.com|mirrors.ustc.edu.cn/proxmox|g' /etc/apt/sources.list.d/pve-enterprise.list
# 修改 Ceph 源
sed -i 's|enterprise.proxmox.com|mirrors.ustc.edu.cn/proxmox|g' /etc/apt/sources.list.d/ceph.list
# 修改 PVE 和 Ceph 为非订阅源
sed -i 's|enterprise|no-subscription|g' /etc/apt/sources.list.d/{pve-enterprise,ceph}.list
```

去除未订阅提示：

```xml
sed -i.bak "s/data.status.toLowerCase() !== 'active'/false/g" /usr/share/javascript/proxmox-widget-toolkit/proxmoxlib.js
systemctl restart pveproxy.service
```

## 参考链接

1. [Proxmox VE 下载](https://www.proxmox.com/en/downloads)
2. [Proxmox VE HA 要求](https://pve.proxmox.com/pve-docs/chapter-ha-manager.html#_requirements)