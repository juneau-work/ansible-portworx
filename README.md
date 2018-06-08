## 部署前 ##
1. 己安装好centos 7操作系统
2. 准备ansible环境

``` shell
# 安装pip
yum -y install python-setuptools
easy_install pip
# 安装ansible
pip install ansible
```

3. 配置主机列表
**dev/hosts**  
> etcd是etcd集群节点组
> px-enterprise是px-enterprise集群节点组
> px-lighthouse是px web ui节点,该节点只要安装好docker即可
``` ini
[etcd]
192.168.130.11
192.168.130.12
192.168.130.13

[px-enterprise]
192.168.130.11
192.168.130.12
192.168.130.13

[px-lighthouse]
192.168.130.11
```
4. 定义变量
**dev/group_vars/all**
``` yaml
portworx_enterprise_version: 1.2.22
portworx_disks: /dev/sdb,/dev/sdc # 多块磁盘或分区用逗号隔开,如/dev/sdb,/dev/sdc1,/dev/sdc2
portworx_cluster_id: 66c6a195-99c4-46a8-b831-39079269ec2c # 通过uuidgen命令生成
portworx_manage_interface: ens33 # 管理流量网卡
portworx_data_interface: ens33 # 数据流量网卡

dce_offline_repo: http://192.168.130.1:15000/repo/centos-7.4.1708 # dce离线yum源,提供docker,ntp等软件包
hub_offline_prefix: 192.168.130.1:5000/daocloud # px离线镜像仓库，可以用dce内建镜像仓库，也可以自行搭建，甚至可以直接将镜像push到dce离线源中来作为px离线镜像仓库。dce-plugin-proxy,influxdb,px-lighthouse都使用latest版本
```
> **注意:** 对于敏感数据，如远程用户名密码, 请事先通过以下脚本生成密文
``` shell
cat <<'EOF' > vault.sh
VAULT_ID='myVAULT@2018'
echo $VAULT_ID > ~/.vault_pass.txt

ANSIBLE_USER='root'
ANSIBLE_PASSWORD='root'

ansible-vault encrypt_string --vault-id ~/.vault_pass.txt $ANSIBLE_USER --name 'vault_ansible_user' | tee dev/group_vars/vault
ansible-vault encrypt_string --vault-id ~/.vault_pass.txt $ANSIBLE_PASSWORD --name 'vault_ansible_password' | tee -a dev/group_vars/vault
EOF

bash vault.sh
```





-------------------------------------------------------------------------------
## 安装px ##
> ansible-playbook -i dev/hosts --vault-password-file ~/.vault_pass.txt one_step_install.yml --extra-vars install_or_uninstall=install





-------------------------------------------------------------------------------
## 卸载px ##
> ansible-playbook -i dev/hosts --vault-password-file ~/.vault_pass.txt one_step_uninstall.yml --extra-vars install_or_uninstall=uninstall





-------------------------------------------------------------------------------
## 升级px ##
> ansible-playbook -i dev/hosts --vault-password-file ~/.vault_pass.txt upgrade.yml
