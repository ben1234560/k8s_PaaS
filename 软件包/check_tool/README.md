使用方式：
yum install ansible -y  # 下载ansible

手动做当前机器与其它机器的免密登陆，如当前机器ssh登陆到全部节点（共个）

ansible-playbook -i hosts main.yml


