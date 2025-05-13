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

