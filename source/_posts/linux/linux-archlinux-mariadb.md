---
title: 如何在Arch Linux上面安装MariaDB？
date: 2023-12-22 20:06:06
category: Linux
tags: 
    - ChatGPT
---
这是一个GPT4的回答，之前一直使用ubuntu系统，众所周知，在debian系统上面安装mysql这样的软件一行命令就可以，超级简单。

但是Arch不行，默认情况下它居然连初始化的工作都没做，更不要说安装服务了，于是问了下ChatGPT，答案非常好！


在 Arch Linux 上安装 MariaDB 的步骤通常如下：

1. **更新系统包数据库**：
   首先，确保您的系统是最新的。打开终端并运行以下命令来更新系统包数据库和已安装的包：
   ```bash
   sudo pacman -Syu
   ```

2. **安装 MariaDB**：
   接下来，使用 pacman 包管理器安装 MariaDB。在终端中运行以下命令：
   ```bash
   sudo pacman -S mariadb
   ```

3. **初始化数据库系统**：
   MariaDB 安装完成后，您需要初始化数据库系统。运行以下命令来执行这个步骤：
   ```bash
   sudo mariadb-install-db --user=mysql --basedir=/usr --datadir=/var/lib/mysql
   ```
   
<!--more-->

4. **启动 MariaDB 服务**：
   使用 systemd 启动 MariaDB 服务。运行以下命令来启动服务并设置开机自启动：
   ```bash
   sudo systemctl start mariadb
   sudo systemctl enable mariadb
   ```

5. **安全配置**：
   运行 MariaDB 的安全脚本来设置根用户密码，移除匿名用户，禁止根用户远程登录，以及删除测试数据库。运行以下命令：
   ```bash
   sudo mysql_secure_installation
   ```
   按照提示进行操作，设置密码和其他安全选项。

6. **测试安装**：
   最后，您可以通过运行以下命令来测试 MariaDB 是否已正确安装和配置：
   ```bash
   mysql -u root -p
   ```
   输入您在安全配置过程中设置的密码。如果能够成功登录，说明 MariaDB 已经正确安装和配置。

请注意，以上步骤可能会根据 MariaDB 和 Arch Linux 的新版本进行变更。始终建议查阅最新的官方文档以获得最准确的安装指导。