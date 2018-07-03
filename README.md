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
> px-client是最终使用px集群的客户端，如果px集群和dce集群所在主机相同，则无需重复安装px-client;当px集群完全独立于dce集群时，则dce集群节点全部要装px-client,也就是需要把dce集群所有节点都添加到px-client章节
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

[px-client]
```
4. 定义变量
**dev/group_vars/all**
``` yaml
portworx_enterprise_version: 1.2.22
portworx_disks: /dev/sdb,/dev/sdc # 多块磁盘或分区用逗号隔开,如/dev/sdb,/dev/sdc1,/dev/sdc2
portworx_cluster_id: 66c6a195-99c4-46a8-b831-39079269ec2c # 通过uuidgen命令生成
portworx_manage_interface: ens33 # 管理流量网卡
portworx_data_interface: ens33 # 数据流量网卡


centos7_base_repo: http://192.168.130.1/ftp/centos7 # centos7 base源, 用来安装px依赖kernel-devel,kernel-headers
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
> 注: 如果已经安装好了dce，则需要跳过hostname,hosts,docker  
> ansible-playbook -i dev/hosts --vault-password-file ~/.vault_pass.txt one_step_install.yml --extra-vars install_or_uninstall=install --skip-tags hostname,hosts,docker





-------------------------------------------------------------------------------
## 卸载px ##
> ansible-playbook -i dev/hosts --vault-password-file ~/.vault_pass.txt one_step_uninstall.yml --extra-vars install_or_uninstall=uninstall





-------------------------------------------------------------------------------
## 升级px ##
> ansible-playbook -i dev/hosts --vault-password-file ~/.vault_pass.txt upgrade.yml
   
   
   
   
   
-------------------------------------------------------------------------------
## 错误解决 ##
#### 1. The following packages have pending transactions ####
报错:
```txt
failed: [192.168.130.11] (item=[u'kernel-devel', u'kernel-headers']) => {"changed": false, "item": ["kernel-devel", "kernel-headers"], "msg": "The following packages have pending transactions: kernel-headers-x86_64", "rc": 125, "results": ["kernel-devel-3.10.0-693.el7.x86_64 providing kernel-devel is already installed"]}  
```
原因:  网络不稳定或人为等原因造成yum安装包的事务没走完，需要清理掉未完成安装的事务   
解决: 
> ansible -i dev/hosts all --vault-password-file ~/.vault_pass.txt -e @dev/group_vars/vault -m shell -a 'yum -y install yum-utils && yum-complete-transaction --cleanup-only'
