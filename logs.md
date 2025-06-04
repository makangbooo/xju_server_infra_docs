## 2025.6.2
重启服务器
```bash
# 1. scancel 节点
# 2. 重启命令
sudo reboot
# 3. 安装驱动
cd management
sudo bash NVIDIA-Linux-x86_64-550.127.05.run
# 4. NFS 挂载
mount -a
# 5. slurm
sudo systemctl restart slurmd
sudo scontrol update nodename=xju-aslp5 state=resume
```

## 2025.5.27

为某个文件增加访问权限。要求：
- 仅指定组的组员访问该文件夹
- 兼容nfs磁盘共享

```bash
# 1.创建属组（s5、s6）
sudo groupadd -g 168001 xjudata
getent group xjudata # 查看属组信息

# 2.设置文件夹访问权限
ls -ld # 查看当前文件夹权限
sudo chgrp -R xjudata data # 改变data文件夹属组为xjudata
sudo chmod g+s data # 对改新文件夹的新增文件要求继承data的属组

# 3.为xjudata属组增加用户（s5、s6都要设置）
sudo usermod -aG xjudata nur
sudo usermod -aG xjudata mt522
sudo usermod -aG xjudata yhl522
sudo usermod -aG xjudata mkb524

# 4.验证：通过组名查看其成员
getent group xjudata

# 5.为路径添加权限限制（仅属组内的成员可以读写）（s6）
sudo chgrp -R xjudata /s6home/mt522
sudo chmod -R 750 /s6home/mt522
# 第一位（7） → 所有者（user）权限
# 第二位（7） → 同组用户（group）权限
# 第三位（0） → 其他人（others）权限
# 770：所有者：rwx，组用户：rwx，其他人：无权限
# 750：所有者：rwx，组用户：r-x，其他人：无权限

```

## 2025.5.26

装显卡。要求：
- 重装驱动
- slurm恢复

```bash
# 1.关机
sudo shutdown

# 2.拆机

# 3.显卡驱动安装
bash /home/huang/management/NVIDIA-Linux-x86_64-550.127.05.run

# 4.slurm恢复
sudo scontrol update nodename=<nodename> state=resume

```



## 2025.5.15

管理员创建用户流程

创建用户的脚本：`/home/huang/management/new_user.sh`

```bash
#!/bin/bash
name=$1
id=$2
homedir=$3
passwd='$6$ltByxkgm$nGGTxpW.bgfOwqRY7fY83aJuookhyqdMf0/OoO5ulLRgiaDtSArhxAfTsM0ZDxL9NY7eJcxoqqKAc6s5MlBxJ/'
groupadd -g ${id} ${name}
useradd -d /home/${name} -M -u ${id} -g ${id} -s /bin/bash -p $passwd ${name}
mkdir /${homedir}/${name}
find /etc/skel/ -maxdepth 1 -name ".*" -exec cp -r {} /${homedir}/${name} \;
chown -R ${name}:${name} /${homedir}/${name}
setquota -u ${name} 512G 512G 0 0 /${homedir}
ln -s /${homedir}/${name} /home/${name}
echo "sudo groupadd -g ${id} ${name} && sudo useradd -d /home/${name} -M -u ${id} -g ${id} -s /bin/bash -p $passwd ${name} && sudo ln -s /${homedir}/${name} /home/${name}"
```

### 举个例子

为创建zsb524账号

- 1. 执行sudo ./new_user.sh zsb524 1000013 s6home2

  ```bash
  huang@xju-aslp5:~/management$ sudo ./new_user.sh zsb524 1000013 s6home2
  mkdir: cannot create directory ‘/s6home2/zsb524’: File exists
  setquota: Mountpoint (or device) /s6home2 not found or has no quota enabled.
  setquota: Not all specified mountpoints are using quota.
  sudo groupadd -g 1000013 zsb524 && sudo useradd -d /home/zsb524 -M -u 1000013 -g 1000013 -s /bin/bash -p $6$ltByxkgm$nGGTxpW.bgfOwqRY7fY83aJuookhyqdMf0/OoO5ulLRgiaDtSArhxAfTsM0ZDxL9NY7eJcxoqqKAc6s5MlBxJ/ zsb524 && sudo ln -s /s6home2/zsb524 /home/zsb524
  ```



- 2. 将echo出来的，在slurm、nfs集群的各机器中都执行一遍
     ```bash
     sudo groupadd -g 1000013 zsb524 && sudo useradd -d /home/zsb524 -M -u 1000013 -g 1000013 -s /bin/bash -p $6$ltByxkgm$nGGTxpW.bgfOwqRY7fY83aJuookhyqdMf0/OoO5ulLRgiaDtSArhxAfTsM0ZDxL9NY7eJcxoqqKAc6s5MlBxJ/ zsb524 && sudo ln -s /s6home2/zsb524 /home/zsb524
     ```





## 2025.5.13

为用户设置分组，设置对某个路径下文件的读写操作

```bash
# 1.创建属组（s5、s6）
sudo groupadd -g 168001 xjudata
# 2.为属组增加用户（s5、s6）
sudo usermod -aG xjudata nur
sudo usermod -aG xjudata mt522
sudo usermod -aG xjudata yhl522
sudo usermod -aG xjudata mkb524
# 3.验证：通过组名查看其成员
getent group xjudata

# 4.为路径添加权限限制（仅属组内的成员可以读写）（s6）
sudo chgrp -R xjudata /s6home/mt522
sudo chmod -R 750 /s6home/mt522 
# 第一位（7） → 所有者（user）权限
# 第二位（7） → 同组用户（group）权限
# 第三位（0） → 其他人（others）权限
# 770：所有者：rwx，组用户：rwx，其他人：无权限
# 750：所有者：rwx，组用户：r-x，其他人：无权限


```

