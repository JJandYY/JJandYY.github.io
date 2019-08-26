---
layout: post
title: "ubuntu中安装docker"
data: 2019-01-02 20:50:22 +0800
author: "YY"
tags:
    - 总结
    - Docker
---

## 在Ubuntu中安装Docker

更新ubuntu的apt源索引

```python
sudo apt-get update
```

安装包允许apt通过HTTPS使用仓库

```python
sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    software-properties-common
```

添加Docker官方GPG key

```python
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```

设置Docker稳定版仓库

```python
sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
```

添加仓库后，更新apt源索引

```python
sudo apt-get update
```

安装最新版Docker CE（社区版）

```python
sudo apt-get install docker-ce
```

检查Docker CE是否安装正确

```python
sudo docker run hello-world
```

出现Hello from Docker！说明安装成功。

为了避免每次命令都输入sudo，可以设置用户权限，**注意执行后须注销重新登录**

```python
sudo usermod -a -G docker $USER
```