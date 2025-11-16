# Snort3 入侵检测系统自动化部署项目

[![Ansible](https://img.shields.io/badge/Ansible-Automation-red)](https://www.ansible.com/)
[![Snort3](https://img.shields.io/badge/Snort3-IDS-blue)](https://www.snort.org/)
[![License](https://img.shields.io/badge/License-MIT-green)](LICENSE)

## 项目概述

本项目通过 Ansible Playbook 实现 Snort3 入侵检测系统的全自动化部署与配置。采用模块化设计，支持灵活配置，能够快速在生产环境中部署功能完整的 Snort3 IDS/IPS 系统。

## 核心设计理念

### 1. 基础设施即代码 (Infrastructure as Code)
- 所有配置通过 YAML 文件定义，确保部署的一致性和可重复性
- 版本控制跟踪配置变更，便于审计和回滚

### 2. 分层架构设计
```
应用层 (Snort3) -> 配置层 (Lua配置) -> 依赖层 (LibDAQ) -> 系统层 (依赖包)
```

### 3. 幂等性保证
- 每个任务都设计为可重复执行
- 状态检查避免重复操作
- 条件执行确保资源高效利用

## Playbook 架构解析

### 依赖管理层
```yaml
- 安装编译工具链 (gcc-c++, cmake, autoconf等)
- 安装网络开发库 (libpcap-devel, libnfnetlink-devel等)
- 安装高性能库 (hyperscan, gperftools-devel)
```

### 核心组件层
1. **LibDAQ 数据采集库**
   - 从官方仓库编译安装
   - 环境变量优化配置
   - 动态链接库配置

2. **Snort3 主程序**
   - 源码编译安装，优化性能参数
   - 符号链接创建，确保命令行可用性
   - 多阶段验证安装结果

### 配置管理层
```yaml
目录结构:
/usr/local/snort/
├── bin/snort          # 主程序
├── etc/snort/         # 配置文件
├── etc/rules/         # 规则文件
└── lib/openappid/     # 应用识别库
```

## 关键技术特性

### 1. 智能网络配置
```yaml
- 自动设置网卡混杂模式
- 网络卸载功能检查
- systemd 服务持久化配置
```

### 2. 配置验证机制
```lua
-- Lua 配置文件语法验证
- 使用 dofile() 替代 require() 避免路径问题
- 实时语法检查与错误报告
- 优雅降级处理配置错误
```

### 3. 规则管理
```yaml
- 社区规则自动下载与部署
- 本地自定义规则支持
- OpenAppID 应用识别扩展
```

### 4. 服务集成
```systemd
# 网络接口服务
snort3-nic.service -> 确保网卡配置

# 主服务
snort3.service -> Snort3 守护进程管理
```

## 部署流程

### 阶段 1: 环境准备
1. 系统依赖包安装
2. CMake 版本验证
3. 专用用户和目录创建

### 阶段 2: 核心组件构建
1. LibDAQ 编译安装
2. Snort3 源码编译
3. 性能优化参数配置

### 阶段 3: 配置部署
1. Lua 配置文件部署
2. 网络规则配置
3. 系统服务集成

### 阶段 4: 验证测试
1. 配置语法验证 (`snort -T`)
2. 服务功能测试
3. 清理临时文件

## 配置说明

### 主要变量文件 (`all.yml`)
```yaml
snort_install_dir: "/usr/local/snort"
interface: "eth0"
home_net: "192.168.1.0/24"
community_rules_url: "https://www.snort.org/downloads/community/community-rules.tar.gz"
```

### 库存配置 (`inventory.ini`)
支持多环境部署：
- `hosts`: 远程目标主机
- `local`: 本地测试环境

## 安全特性

- 专用非特权用户运行 (`snort`用户)
- 最小权限目录访问控制
- 安全配置文件权限 (644/755)
- 网络隔离配置

## 监控与验证

### 安装验证点
1. **二进制验证**: `snort --version`
2. **配置测试**: `snort -c config -T`
3. **服务状态**: systemd 服务监控
4. **网络配置**: 混杂模式确认

### 日志记录
- 结构化日志输出
- 错误分级处理
- 详细调试信息

## 项目优势

### 1. 自动化程度高
- 一键式部署，无需人工干预
- 自动依赖解析和安装
- 智能错误恢复机制

### 2. 可维护性强
- 模块化任务设计
- 清晰的变量管理
- 详细的文档说明

### 3. 生产就绪
- 健壮的错误处理
- 完整的验证流程
- 服务化集成

### 4. 灵活扩展
- 易于定制规则和配置
- 支持多环境部署
- 可扩展的模板系统

## 文件结构
```
.
├── all.yml                 # 全局变量配置
├── ansible.cfg            # Ansible 配置
├── inventory.ini          # 库存文件
├── snort.yml              # 主 Playbook
├── templates/             # 配置模板
│   ├── snort3-nic.service.j2
│   └── snort3.service.j2
└── README.md              # 项目文档
```

## 使用方法

### 前置要求

#### 1. 系统要求
- 操作系统: RHEL/CentOS 8/9 或 Fedora
- 内存: 至少 2GB RAM
- 磁盘空间: 至少 5GB 可用空间
- 网络: 需要访问 GitHub 和 Snort 官网

#### 2. 软件要求
- Ansible 2.9+
- Python 3.6+
- 目标主机需要支持 SSH 连接

#### 3. 权限要求
- 目标主机需要 sudo 权限
- 控制机需要 SSH 密钥或密码访问目标主机

### 快速开始

#### 步骤 1: 下载项目
您可以通过两种方式获取项目代码：

**方式一：直接下载发布版本（推荐）**
```bash
# 下载最新发布版本
wget https://github.com/Prism-ywddc/ansible_install_snort/archive/refs/tags/v1.0.tar.gz

# 解压文件
tar -xzf v1.0.tar.gz

# 进入项目目录
cd ansible_install_snort-1.0
```

**方式二：使用 Git 克隆**
```bash
# 克隆项目
git clone https://github.com/Prism-ywddc/ansible_install_snort.git
cd ansible_install_snort

# 切换到稳定版本（可选）
git checkout v1.0
```

#### 步骤 2: 配置库存文件
编辑 `inventory.ini` 文件，添加目标主机：
```ini
[hosts]
snort-server ansible_host=192.168.1.100 ansible_user=admin ansible_ssh_private_key_file=~/.ssh/id_rsa

# 或者使用密码认证
# snort-server ansible_host=192.168.1.100 ansible_user=admin ansible_ssh_pass=your_password
```

#### 步骤 3: 调整配置变量
编辑 `group_vars/all.yml` 文件，根据你的环境修改关键配置：
```yaml
# 网络接口配置
interface: "eth0"                    # 监控的网络接口
home_net: "192.168.1.0/24"          # 受保护的网络段

# 安装路径配置
snort_install_dir: "/usr/local/snort"
snort_config_dir: "{{ snort_install_dir }}/etc/snort"
snort_rules_dir: "{{ snort_install_dir }}/etc/rules"

# 用户配置
snort_user: "snort"
snort_group: "snort"
```

#### 步骤 4: 测试连接
```bash
# 测试 Ansible 连接
ansible -i inventory.ini hosts -m ping

# 测试权限
ansible -i inventory.ini hosts -m shell -a "whoami" -b
```

#### 步骤 5: 执行部署
```bash
# 完整部署
ansible-playbook snort.yml

# 详细输出（推荐用于调试）
ansible-playbook snort.yml -v

# 非常详细的输出
ansible-playbook snort.yml -vvv
```

#### 步骤 6: 验证安装
```bash
# 在目标主机上验证 Snort 安装
ssh your-server
/usr/local/snort/bin/snort --version

# 测试配置
/usr/local/snort/bin/snort -c /usr/local/snort/etc/snort/snort.lua -T

# 检查服务状态
systemctl status snort3
systemctl status snort3-nic
```

### 高级用法

#### 1. 自定义规则配置
编辑 `group_vars/all.yml` 文件，添加自定义规则：
```yaml
custom_rules:
  - "alert tcp any any -> $HOME_NET 22 (msg:\"SSH Brute Force Attempt\"; flow:established,to_server; threshold:type threshold, track by_src, count 5, seconds 60; sid:1000002; rev:1;)"
  - "alert http any any -> $HOME_NET any (msg:\"Potential SQL Injection\"; http.uri; content:\"union\"; nocase; content:\"select\"; nocase; sid:1000003; rev:1;)"
```

#### 2. 多环境部署
创建不同的库存文件：
```bash
# 生产环境
ansible-playbook -i production.ini snort.yml

# 测试环境
ansible-playbook -i staging.ini snort.yml

# 开发环境
ansible-playbook -i development.ini snort.yml
```

#### 3. 标签执行
使用标签执行特定任务：
```bash
# 只安装依赖
ansible-playbook snort.yml --tags "dependencies"

# 只配置网络
ansible-playbook snort.yml --tags "network"

# 只部署规则
ansible-playbook snort.yml --tags "rules"

# 跳过验证
ansible-playbook snort.yml --skip-tags "validation"
```

#### 4. 变量覆盖
在命令行中覆盖变量：
```bash
ansible-playbook snort.yml -e "interface=ens192 home_net=10.0.0.0/24"
```

### 故障排除

#### 常见问题

1. **编译失败**
   - 检查网络连接，确保可以访问 GitHub
   - 验证系统有足够的内存和磁盘空间
   - 查看详细的错误日志：`ansible-playbook snort.yml -vvv`

2. **配置测试失败**
   - 检查 Lua 配置文件语法
   - 验证规则文件完整性
   - 查看 Snort 错误输出：`/usr/local/snort/bin/snort -c /usr/local/snort/etc/snort/snort.lua -T`

3. **服务启动失败**
   - 检查网卡名称是否正确
   - 验证用户权限
   - 查看系统日志：`journalctl -u snort3`

#### 调试技巧

1. **启用详细输出**
```bash
ansible-playbook snort.yml -vvv
```

2. **检查特定任务**
```bash
ansible-playbook snort.yml --start-at-task "安装 Snort"
```

3. **手动验证步骤**
```bash
# 在目标主机上手动运行
cd /tmp/snort3
./configure_cmake.sh --prefix=/usr/local/snort --enable-tcmalloc
cd build && make -j4
```

### 维护操作

#### 更新规则
```bash
# 手动更新社区规则
ansible-playbook snort.yml --tags "rules"

# 或者直接运行规则更新任务
ansible -i inventory.ini hosts -m get_url -a "url={{ community_rules_url }} dest=/tmp/snort3-community-rules.tar.gz" -b
```

#### 重启服务
```bash
# 通过 Ansible 重启
ansible -i inventory.ini hosts -m systemd -a "name=snort3 state=restarted" -b

# 或者在目标主机上
systemctl restart snort3
systemctl restart snort3-nic
```

#### 查看日志
```bash
# Snort 应用日志
tail -f /var/log/snort/alert.csv

# 系统服务日志
journalctl -u snort3 -f
```

### 卸载说明

要完全卸载 Snort3，可以执行以下步骤：

```bash
# 停止服务
ansible -i inventory.ini hosts -m systemd -a "name=snort3 state=stopped" -b
ansible -i inventory.ini hosts -m systemd -a "name=snort3-nic state=stopped" -b

# 删除服务文件
ansible -i inventory.ini hosts -m file -a "path=/etc/systemd/system/snort3.service state=absent" -b
ansible -i inventory.ini hosts -m file -a "path=/etc/systemd/system/snort3-nic.service state=absent" -b

# 删除安装目录
ansible -i inventory.ini hosts -m file -a "path={{ snort_install_dir }} state=absent" -b

# 删除日志目录
ansible -i inventory.ini hosts -m file -a "path=/var/log/snort state=absent" -b

# 删除用户
ansible -i inventory.ini hosts -m user -a "name=snort state=absent" -b
```

## 下载和安装说明

### 获取项目代码

#### 方法一：下载发布版本（稳定推荐）
```bash
# 下载最新发布版本 v1.0
wget https://github.com/Prism-ywddc/ansible_install_snort/archive/refs/tags/v1.0.tar.gz

# 验证文件完整性（可选）
sha256sum v1.0.tar.gz

# 解压文件
tar -xzf v1.0.tar.gz

# 进入项目目录
cd ansible_install_snort-1.0

# 查看文件结构
ls -la
```

#### 方法二：Git 克隆（开发版本）
```bash
# 克隆项目仓库
git clone https://github.com/Prism-ywddc/ansible_install_snort.git

# 进入项目目录
cd ansible_install_snort

# 切换到稳定版本（可选）
git checkout v1.0
```

#### 方法三：使用 curl 下载
```bash
# 使用 curl 下载
curl -L -o snort-ansible-v1.0.tar.gz https://github.com/Prism-ywddc/ansible_install_snort/archive/refs/tags/v1.0.tar.gz

# 解压
tar -xzf snort-ansible-v1.0.tar.gz

# 进入目录
cd ansible_install_snort-1.0
```

### 验证下载完整性

下载完成后，建议验证文件完整性：

```bash
# 检查文件大小
ls -lh v1.0.tar.gz

# 计算 SHA256 校验和（可选）
sha256sum v1.0.tar.gz
# 预期输出: [文件校验和值]

# 解压并检查文件结构
tar -tzf v1.0.tar.gz | head -20
```

### 项目结构验证

解压后，您应该看到以下文件结构：
```
ansible_install_snort-1.0/
├── ansible.cfg
├── inventory.ini
├── snort.yml
├── group_vars/
│   └── all.yml
└── templates/
    ├── snort3-nic.service.j2
    └── snort3.service.j2
```

## 许可证

本项目采用 MIT 许可证 - 详见 LICENSE 文件。

---

**让网络安全部署变得简单高效！**
