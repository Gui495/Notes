## 1. 文件说明
- 该文件是 CentOS 7 的 yum 仓库配置文件，使用华为云镜像 `repo.huaweicloud.com`。
- 文件通常保存为 `/etc/yum.repos.d/CentOS-Base.repo` 或 `CentOS-7-reg.repo`。
- 作用：配置 yum 软件源，使 CentOS 7 可以从华为云镜像下载和更新软件包。

## 2. 仓库配置
### 2.1 base 仓库
```ini
[base]
name=CentOS-$releasever - Base - repo.huaweicloud.com
baseurl=https://repo.huaweicloud.com/centos/$releasever/os/$basearch/
#mirrorlist=https://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=os
gpgcheck=1
gpgkey=https://repo.huaweicloud.com/centos/RPM-GPG-KEY-CentOS-7
```
- `[base]`：仓库 ID。
- `name`：仓库名称。
- `baseurl`：华为云镜像地址，`$releasever` 为系统版本，`$basearch` 为系统架构。
- `mirrorlist`：官方镜像列表，已注释。
- `gpgcheck=1`：启用 GPG 校验。
- `gpgkey`：GPG 公钥地址。

### 2.2 updates 仓库
```ini
[updates]
name=CentOS-$releasever - Updates - repo.huaweicloud.com
baseurl=https://repo.huaweicloud.com/centos/$releasever/updates/$basearch/
gpgcheck=1
gpgkey=https://repo.huaweicloud.com/centos/RPM-GPG-KEY-CentOS-7
```
- 用于系统更新包。

### 2.3 extras 仓库
```ini
[extras]
name=CentOS-$releasever - Extras - repo.huaweicloud.com
baseurl=https://repo.huaweicloud.com/centos/$releasever/extras/$basearch/
gpgcheck=1
gpgkey=https://repo.huaweicloud.com/centos/RPM-GPG-KEY-CentOS-7
```
- 额外软件包，通常默认启用。

### 2.4 centosplus 仓库
```ini
[centosplus]
name=CentOS-$releasever - Plus - repo.huaweicloud.com
baseurl=https://repo.huaweicloud.com/centos/$releasever/centosplus/$basearch/
gpgcheck=1
enabled=0
gpgkey=https://repo.huaweicloud.com/centos/RPM-GPG-KEY-CentOS-7
```
- `enabled=0`：默认不启用。
- 包含增强功能软件包，可能替换基础包，谨慎启用。

## 3. 常用命令
更新 yum 缓存：
```bash
yum clean all
yum makecache
```
查看仓库：
```bash
yum repolist
```
安装软件：
```bash
yum install 软件名 -y
```
更新系统：
```bash
yum update -y
```

## 4. 注意事项
- `$releasever` 和 `$basearch` 由 yum 自动替换，不要手动写死。
- `gpgcheck=1` 建议保持开启，确保软件包来源可信。
- 如果华为云镜像不可用，可取消 `mirrorlist` 注释或更换其他镜像源。
- `centosplus` 默认 `enabled=0`，如无特殊需求不要启用。
- 修改仓库文件后执行 `yum clean all && yum makecache` 使配置生效。
- CentOS 7 已停止维护，生产环境需考虑迁移或使用 vault 源。

## 5. 小结
- 该文件配置了 CentOS 7 的 base、updates、extras、centosplus 四个仓库。
- 使用华为云镜像加速 yum 下载。
- base/updates/extras 默认启用，centosplus 默认禁用。
- 通过 `yum clean all`、`yum makecache`、`yum repolist` 验证配置。