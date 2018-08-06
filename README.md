# JupyterHub-setup
该文档描述了如何使用云服务器来建立jupyterHub 方便多人使用jupyter。  
注意：
1. 请开启确保云服务器商的网络安全组对应端口打开，避免被拦截流量外部无法访问。 
2. 通过服务器的监控面板了解访问的原因，是内存/cpu/带宽。我最开始使用的最低配置 1核 1G 1M 带宽，访问慢的主要原因是带宽不够，在使用按流量付费后，访问速度是秒开。   
3. jupyterhub 所使用的账号系统及 linux系统本身所自带的账号。批量创建账号时要确保这些账号能够有家目录。

//批量创建用户
```
seq -w 20|sed -r "s#(.*)#useradd udacity_vip\1 -m  #g"|bash
tail -20 /etc/passwd
echo udacity_vip{01..20}:$((RANDOM))|tr " " "\n" >pass.log
chpasswd <pass.log
```

//root 身份登录
```bash
apt-get update
apt-get upgrade
wget https://mirrors.tuna.tsinghua.edu.cn/anaconda/archive/Anaconda3-5.1.0-Linux-x86_64.sh
bash Anaconda3-5.1.0-Linux-x86_64.sh
```
//配置新的清华源,加快包的下载速度
```bash
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main/
conda config --set show_channel_urls yes
```
//修改anaconda 为所有人可读、执行
```
chmod a+rx -r /root/Anaconda3
```
//立即生效配置 下载jupyterhub
```
source ./bashrc
conda install -c conda-forge jupyterhub
conda install notebook
```

//建立jupyterhun文件夹，存放相关配置、输出文件
```
mkdir /etc/jupyterhub
cd /etc/jupyterhub
```

//配置jupyterhub.sh 调用所使用的配置参数
```
jupyterhub --generate-config
echo jupyterhub -f /etc/jupyterhub/jupyterhub_config.py > jupyterhub.sh
echo su -l root /etc/jupyterhub/jupyterhub.sh \& >> /etc/rc.local
```


//修改jupyterhub配置文件 jupyterhub_config.py
```
c.Authenticator.whitelist = {'testuser'}
c.JupyterHub.admin_users = { 'testuser' }
c.JupyterHub.ip = '0.0.0.0'
c.JupyterHub.port = 5888
c.Spawner.notebook_dir = '~/'
c.JupyterHub.ssl_key = '/etc/jupyterhub/ssl/ssl.key'
c.JupyterHub.ssl_cert = '/etc/jupyterhub/ssl/ssl.crt'
```
//创建https链接 所需文件
```
mkdir /etc/jupyterhub/ssl
cd /etc/jupyterhub/ssl
openssl x509 -req -days 365 -in ssl.csr -signkey ssl.key -out ssl.crt
```
//后台运行jupyterhub 
```
nohup jupyterhub > jupyterhub.log &
```


