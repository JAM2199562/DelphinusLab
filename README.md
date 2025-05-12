# DelphinusLab

## 配置要求
* 显卡: RTX 4090以上
* 内存: >82 GB以上
* 软件：Nvidia CUDA, Docker
* 系统：Ubuntu 或者Windows安装Ubuntu


# 安装

## 第一步: 删除保留变量

* 打开 `/etc/environment`:
```bash
sudo nano /etc/environment
```
删除所有内容，添加以下代码：
```
PATH="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
```
* ctrl+O保存，回车，ctrl+x退出
## 第二部: 安装和更新软件包
```bash
sudo apt-get update && sudo apt-get upgrade -y
```
```bash
sudo apt install curl iptables build-essential git wget lz4 jq make gcc nano automake autoconf tmux htop nvme-cli libgbm1 pkg-config libssl-dev libleveldb-dev tar clang bsdmainutils ncdu unzip libleveldb-dev  -y
```

## 第三步: 安装 Docker
```bash
sudo apt update -y && sudo apt upgrade -y
for pkg in docker.io docker-doc docker-compose podman-docker containerd runc; do sudo apt-get remove $pkg; done

sudo apt-get update
sudo apt-get install ca-certificates curl gnupg
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

echo \
  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update -y && sudo apt upgrade -y

sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Test Docker
sudo docker run hello-world
```

## 第四步: 安装 nVIDIA Cuda
```bash
distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
curl -s -L https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg

curl -s -L https://nvidia.github.io/libnvidia-container/$distribution/libnvidia-container.list | \
  sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#' | \
  sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list > /dev/null

```
```bash
sudo apt-get update
```
```bash
sudo apt install nvidia-cuda-toolkit
```
```bash
sudo apt-get install -y nvidia-container-toolkit
```
```bash
sudo nvidia-ctk runtime configure --runtime=docker --set-as-default
```
```bash
sudo apt install nvidia-cuda-toolkit
```
```bash
sudo rmmod nvidia_drm nvidia_modeset nvidia_uvm nvidia
```
```bash
sudo systemctl enable docker
sudo systemctl restart docker
```

验证安装成功命令：
* Nvidia 驱动: `nvidia-smi`
* Cuda toolkits: `nvcc --version`

## 第五步: 设置 Up HugePages (内存)

* 检查内存：（至少需要 80 GB）
```bash
free -h
```

* 检查当前的 HugePages：
```bash
grep Huge /proc/meminfo
```
确保HugePages_Total至少为 15000。

* 临时设置HugePages：
```bash
sudo sysctl -w vm.nr_hugepages=15000
```

* 永久创建 HugePages：

编辑/etc/sysctl.conf文件：
```bash
sudo nano /etc/sysctl.conf
```
加入代码:
```
vm.nr_hugepages=15000
```
![image](https://github.com/user-attachments/assets/9cf4b872-7c49-48b9-9fb5-aeed5942e984)

* ctrl+O保存，回车，ctrl+x退出

应用配置:
```bash
sudo sysctl -p
```

验证:
```bash
grep Huge /proc/meminfo
```

## 第六步: 构建证明器
* 克隆仓库：
```bash
git clone https://github.com/DelphinusLab/prover-node-docker
```
```bash
cd prover-node-docker
```

* 配置prover_config.json文件：
```bash
nano prover_config.json
```
* 修改第二行server_url和priv_key:  
  * `server_url`:`"https://rpc.zkwasmhub.com:8090"`
  * `priv_key`: 你的EVM钱包私钥 (去掉 `0x` 前缀).建议用空钱包
  * ctrl+O保存，回车，ctrl+x退出

* 配置dry_run_config.json文件
```bash
nano dry_run_config.json
```
* 修改第二行server_url和priv_key:  
  * `server_url`:`"https://rpc.zkwasmhub.com:8090"`
  * `priv_key`: 你的EVM钱包私钥 (去掉 `0x` 前缀).建议用空钱包
  * ctrl+O保存，回车，ctrl+x退出

* 建造: 
```bash
bash scripts/build_image.sh
```

## 第七步: 运行 Prover
* 编辑scripts/start.sh文件：
```
nano scripts/start.sh
```
在最后一行docker compose up后面添加 -d
* ctrl+O保存，回车，ctrl+x退出
  
![c113b36e2fef5fcdbde2b52d6439340](https://github.com/user-attachments/assets/5c872d0d-4bfa-4f69-915a-988af1e34b2a)



* 运行节点:
```bash
bash scripts/start.sh
```

* 查看日志:
```bash
cd prover-node-docker

docker compose logs -fn 100
```

* 附加:
  * 停止节点: `docker compose down -v` or `bash scripts/stop.sh`
  * 重新启动节点: `docker compose up -d`
  * 查看所有docker容器: `docker ps -a`

## 第八步: 检查证明者节点统计数据
打开https://explorer.zkwasmhub.com/nodes
网站输入刚才私钥的钱包地址

# 如果内存达标还提示内存不足
## 方法一
* 1 手动设置 WSL2 最大内存上限为 100GB+
打开C:\Users\你的用户名\ 编辑 .wslconfig文件。如果看不到名为 .wslconfig 的文件，就新建一个：
用记事本打开它，内容全部替换为：
```bash
[wsl2]
memory=120GB        # 最大给 WSL2 分配 120GB 内存
processors=20       # 给 WSL2 分配 20 核 CPU
swap=16GB           # 可选：16GB 交换分区
localhostForwarding=true
```
保存并退出
* 2  完全重启 WSL2＆Docker Desktop
  关闭所有 WSL 终端窗口
  在 Windows PowerShell（管理员） 或 普通 PowerShell 执行：
  ```bash
  wsl --shutdown
  ```
 * 3 启动 Docker Desktop
 * 4 再打开一个 WSL2 终端（如 Ubuntu），执行
```bash
 free -h
```

## 方法二
在 Windows 上打开记事本（或你喜欢的编辑器）：

按下 Win + R，输入：

 ```bash
notepad %UserProfile%\.wslconfig
```
粘贴以下内容（自定义设置）：

 ```bash
ini

[wsl2]
memory=128GB
processors=all
swap=16GB
swapFile=C:\\swap-wsl2.vhdx
 ```


保存并关闭记事本。

重启 WSL2（或系统）使其生效：

 ```bash
wsl --shutdown
 ```
然后重新启动你的 Ubuntu 实例。

✅ 验证是否生效
回到 WSL 中运行：

 ```bash
free -h
nproc
```

确认内存和 CPU 核心数量是否变更。
此时你应该能看到内存 total 接近 120GiB

