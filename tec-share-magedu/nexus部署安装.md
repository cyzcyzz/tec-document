## 安装过程

### 部署JDK

```jsx
[root@magedu ~]# yum  -y install java-1.8.0-openjdk #centos或者rocky系统
[root@magedu ~]# apt update && apt -y install openjdk-8-jdk # ubuntu
[root@magedu ~]# java -version #验证安装
```
### 安装nexus

```
#下载文件或者移动文件到指定文件夹
[root@magedu ~]# wget -P /usr/local/src/ xxx
[root@magedu ~]# cp xxx /usr/local/src

# 解压并且创建软连接
[root@magedu ~]# tar xf /usr/local/src/${NEXUS_URL##*/} -C /usr/local
[root@magedu ~]# ln -s /usr/local/nexus-*/ ${INSTALL_DIR}
[root@magedu ~]# ln -s ${INSTALL_DIR}/bin/nexus /usr/bin/

```
### 启动nexus

```jsx
cat   > /lib/systemd/system/nexus.service <<EOF
[Unit]
Description=nexus service
After=network.target

[Service]
Type=forking
LimitNOFILE=65536
ExecStart=${INSTALL_DIR}/bin/nexus start
ExecStop=${INSTALL_DIR}/bin/nexus stop
User=root
#User=nexus
Restart=on-abort

[Install]
WantedBy=multi-user.target

EOF

systemctl daemon-reload 
systemctl enable --now  nexus.service
```

### 获取初始密码

进入安装文件夹下

```jsx
../sonatype-work/nexus3/admin.password 即可找到初始密码
```