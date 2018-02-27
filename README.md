## 部署前准备 ##
1. 己安装好centos 7.3/7.4操作系统
2. 准备ansible环境

``` shell
# 安装pip
yum -y install python-setuptools
easy_install pip
# 安装ansible
pip install ansible
```

3.  配置主机列表**dev/hosts**  
	- seed是种子节点,用来初始化集群,只能是一个ip地址
	- manager是manager节点组
	- worker是worker节点组
4.  定义变量
	- **dev/group_vars/all**
	- **dev/group_vars/vault**
> **注意:** 对于敏感数据，如远程用户名密码及dce认证用户名密码, 请事先通过以下脚本生成密文
``` shell
VAULT_ID='myVAULT@2018'
echo $VAULT_ID > ~/.vault_pass.txt

ANSIBLE_USER='root' # 远程操作用户名
ANSIBLE_PASSWORD='root' # 远程操作用户密码，如果通过密钥认证可以删除该变量及下面对应的加密步骤
DCE_USER='foo' # dce认证用户
DCE_PASSWORD='bar' # dce认证用户密码

ansible-vault encrypt_string --vault-id ~/.vault_pass.txt $ANSIBLE_USER --name 'vault_ansible_user' | tee dev/group_vars/vault
ansible-vault encrypt_string --vault-id ~/.vault_pass.txt $ANSIBLE_PASSWORD --name 'vault_ansible_password' | tee -a dev/group_vars/vault
ansible-vault encrypt_string --vault-id ~/.vault_pass.txt $DCE_USER --name 'vault_dce_user' | tee -a dev/group_vars/vault
ansible-vault encrypt_string --vault-id ~/.vault_pass.txt $DCE_PASSWORD --name 'vault_dce_password' | tee -a dev/group_vars/vault
```

-------------------------------------------------------------------------------


- ### 离线安装(内网拉镜像) ###
