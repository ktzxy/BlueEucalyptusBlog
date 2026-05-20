+++
date = '2026-05-20T09:50:57+08:00'
draft = false
title = 'RedHat_DO374'
slug = "kl27csu4e9"
description = "RedHat_DO374"
summary = "RedHat_DO374"
categories = ["📒考试"]
tags = ["RedHat"]
comments = true
series = ["RedHat 认证考试"]
showSeries = true
+++

# DO374 - Ansible 高级自动化

## 0. 环境准备

```bash
# 还原DO374虚拟机快照之后
rht-vmctl fullreset classroom
rht-vmctl fullreset all
# 注： ens35 是 foundation 的第二块网卡，需要连接外网
rht-external --configure ens35

#上传考试环境文件到教学环境
scp -r "\\192.168.149.250\红帽微职位所有材料\RHCA题库\do374\files" kiosk@192.168.239.159:~

# 进入files 目录
rsync -avP git* student@workstation:~/
scp init.sh student@workstation:~/
scp *.tar.gz student@workstation:~/ 注：第七题用到的文件
scp *.tar.xz student@workstation:~/ 注：第九题用到的文件

ssh student@workstation
bash git_prepare.sh
bash init.sh

如果家里的网络无法解析红帽的域名，需要在workstation上添加8.8.8.8的DNS
#sudo echo “nameserver 8.8.8.8” >> /etc/resolv.conf

# 练习说明：
hub.lab.example.com 172.25.250.6 Ansible Hub node
workstation.lab.example.com 172.25.250.9 Ansible control node
servera.lab.example.com 172.25.250.10 Ansible managed node
serverb.lab.example.com 172.25.250.11 Ansible managed node
serverc.lab.example.com 172.25.250.12 Ansible managed node
controller.lab.example.com 172.25.250.7 Ansible Automation Controller control node
serverd.lab.example.com 172.25.250.13 Ansible Automation Controller managed node
servere.lab.example.com 172.25.250.14 Ansible Automation Controller managed node
serverf.lab.example.com 172.25.250.15 Ansible Automation Controller managed node
1.以用户名/密码： student/student 登录 workstation 练习
2.Git 仓库地址： https://git.lab.example.com 用户名 student，密码 Student@123（考试的时候git的credential已经配置好了，不需要输入密码）
3.Ansible 项目默认使用 devops 作为远程登录用户（考试的时候可能是matthew/catherine/admin/greg作为远程登录用户）
4.Execution Environment 使用 ee-supported-rhel8:latest 镜像已经在init.sh脚本中配置好了navigator配置文件，默认使用这个镜像
5.Ansible Automation Controller 地址： https://controller.lab.example.com. 用户名 admin， 密码 redhat
6.Ansible Hub 地址： https://hub.lab.example.com 用户名 student， 密码 redhat123
注意：需要以admin/redhat登录hub.lab.example.com，将student用户设置为Super User。考试环境
已经完成这个配置，由于是教学环境做模拟，所以我们需要自己做这个步骤。
```

## 1. 为用户配置Git

```bash
在 workstation.lab.example.com 上为 student 用户配置 Git:
 Git 用户名: student
 Git 用户电子邮箱: student@lab.example.com
 默认推送方式: simple
除上述要求外，请勿做任何更改

[student@workstation ~]$ git config --global user.name student
[student@workstation ~]$ git config --global user.email student@lab.example.com
[student@workstation ~]$ git config --global push.default simple
[student@workstation ~]$ git config --global -l
user.name=student
user.email=student@lab.example.com
push.default=simple
```

## 2. 创建用户

```bash
在 https://git.lab.example.com/student/create_users.git 找到 create_users 的 git 项目
    create_users.yml 是该项目的剧本文件名
    在 developer 组中添加用户 greg
    在 dev 组中添加 serverc 节点
 commit 并 push 变更到 git 仓库
除上面列出的以外，请勿进行任何其他更改

#解法
[student@workstation ~]$ git clone https://git.lab.example.com/student/create_users.git
[student@workstation create_users]$ cat user_list.yml
users:
  - name: david
   group: developer
  - name: greg
 group: developer
  - name: fred
 group: manager
  - name: sally
 group: developer
[student@workstation create_users]$ cat inventory
[dev]
servera
serverc
[prod]
serverb

# 如发现未安装ansible-navigator，就安装一下
[student@workstation create_users]$ sudo yum -y install ansible-navigator
[sudo] password for student: student

#登录一下镜像仓库
[student@workstation create_users]$ podman login hub.lab.example.com
Username: admin
Password: 
Login Succeeded!

#解决一下ssh的密钥问题（考试不需要做，但是如果考试环境出了此类问题可以先自己尝试解决，如果解决不了可以举手示意请求考官帮你看下是否环境真的有问题）
[student@workstation create_users]$ eval $(ssh-agent)
[student@workstation create_users]$ ssh-add ~/.ssh/lab_rsa

#测试playbook运行
[student@workstation create_users]$ ansible-navigator run -m stdout create_users.yml

#推送到git仓库
[student@workstation create_users]$ git add .
[student@workstation create_users]$ git commit -m "create user"

#为了避免后面做题每次都需要输入密码，可以增加git的用户名配置以及缓存时间（考试不需要输入密码）
[student@workstation create_users]$ git config --global credential.https://git.lab.example.com.username student
[student@workstation create_users]$ git config --global credential.helper cache --timeout=86400
[student@workstation create_users]$ git push
Password for 'https://student@git.lab.example.com': Student@123
```

## 3. 管理 WEB 服务器

```bash
在 Git 仓库 httpd_alias 中管理的 Web 服务器， 添加别名：
 Git 项目在 https://git.lab.example.com/student/httpd_alias.git
 部署新别名的剧本文件是 install_httpd_alias.yml
 只有在安装别名时，才会重启 httpd 服务器。也就是说，如果已经安装了别名，再运行剧本，则不会重新启动 httpd 服务
 commit 并 push 变更到 git 仓库
除上面列出的以外，请勿进行任何其他更改

#解法
[student@workstation ~]$ git clone https://git.lab.example.com/student/httpd_alias.git

[student@workstation httpd_alias]$ vi install_httpd_alias.yml
---
- name: Add Apache alias
  hosts: prod
  become: yes

  tasks:
    - name: copy alias file
      copy:
        src: alias.conf
        dest: /etc/httpd/conf.d
      notify: restart httpd
  handlers:
    - name: restart httpd
      ansible.builtin.service:
        name: httpd
        state: restarted
      listen: restart httpd
[student@workstation httpd_alias]$ ansible-navigator run -m stdout install_httpd_alias.yml
[student@workstation httpd_alias]$ git add .
[student@workstation httpd_alias]$ git commit -m web
[student@workstation httpd_alias]$ git push
```

## 4. 管理网站内容

```bash
在部署到生产之前， 名为 dev 的 web 服务器用于测试网站内容：
Git 项目 manage_content 包含一个不完整的剧本，名为 manage_content.yml ，用于管理 dev web
服务器的内容。在 Git仓库中( https://git.lab.example.com/student/manage_content.git )
完善该剧本，实现:
 当使用标签 alpha 来运行该剧本时，将生成内容“Que Sera, Sera”并部署到 dev 主机上的/var/www/html/index.html 文件中
 当使用标签 beta 来运行该剧本时，将生成内容“Whatever will be, will be”，并且部署到dev主机的 /var/www/html/index.html 文件中
 如果没有使用以上任何一个标签运行剧本，则在受管主机上既不产生也不保存任何信息
 commit 并 push 变更到 git 仓库
除上面列出的以外，请勿进行任何其他更改

#解法
[student@workstation ~]$ git clone https://git.lab.example.com/student/manage_content.git
[student@workstation manage_content]$ vi manage_content.yml
---
- name: Deploy content
  hosts: dev
  become: yes
  tasks:
    - name: tag alpha
      ansible.builtin.copy:
        content: 'Que Sera, Sera'
        dest: /var/www/html/index.html
      tags:
        - alpha
        - never
    - name: tag beta
      ansible.builtin.copy:
        content: 'Whatever will be, will be'
        dest: /var/www/html/index.html
      tags:
        - beta
        - never
[student@workstation manage_content]$ ansible-navigator run -m stdout manage_content.yml
[student@workstation manage_content]$ ansible-navigator run -m stdout manage_content.yml --tags alpha
[student@workstation manage_content]$ curl servera
[student@workstation manage_content]$ ansible-navigator run -m stdout manage_content.yml --tags beta
[student@workstation manage_content]$ curl servera

[student@workstation manage_content]$ git add .
[student@workstation manage_content]$ git commit -m 'tag'
[student@workstation manage_content]$ git push
```

## 5. Ansible 调优

```bash
按照以下要求更新 Git 仓库( https://git.lab.example.com/student/tune_ansible.git )中的
Ansible 配置文件：
 默认情况下， 禁用事实收集
 最大并发主机连接数为 45
 commit 并 push 变更到 git 仓库
除上面列出的以外，请勿进行任何其他更改

#解法
[student@workstation tune_ansible]$ git clone https://git.lab.example.com/student/tune_ansible.git
[student@workstation tune_ansible]$ vi ansible.cfg
[defaults]
collections_paths = /home/student/mycollections
inventory = ./inventory
remote_user = devops
ask_pass = false

forks=45
gathering=explicit

[privilege_escalation]
become = false
become_method = sudo
become_user = root
become_ask_pass = false
[student@workstation tune_ansible]$ git add .
[student@workstation tune_ansible]$ git commit -m tune
[student@workstation tune_ansible]$ git push
```

## 6. 从列表创建用户

```bash
Git 仓库( https://git.lab.example.com/student/create_users_complex.git )包含以下资
源：
user_information.yml 是一个用户账户清单，该文件包含多字段：
    1. name 字段指定账户的用户名和登录 ID
    2. first 字段指定用户的 first name
    3. middle 字段指定用户的 middle name
    4. last 字段指定用户的 last name;
    5. uid 字段指定账户关联的用户 ID
    inventory 是主机清单文件
不要对以上文件做任何修改
创建剧本，使用以上的文件，实现以下操作：
 剧本文件名为 manage_accounts.yml，在 inventory 规定的主机上运行时，该剧本会根据 user_information.yml 文件内容，使用指定的用户 ID 创建用户账户
 针对每个账户，该剧本会按照以下要求生成随机的 6 位数字密码
    1. 必须使用 SHA-512 对密码进行加密
    2. 密码的纯文本版本和用于生成密码的随机值“salt”必须存储在名为“password-”的文件中，其中是与该帐户关联的用户名。例如，针对名为“frederick”用户，密码和“salt”存储在文件 password-frederick 中
 3. 剧本需要在其运行的目录中生成 password-文件
  针对每个账户， user comment (GECOS)字段需要按照以下格式要求为用户设置恰当的名字： First Middle Last(中间有单空格符)，如上所示，名字的每个组成部分都必须大写
 commit 并 push 变更到 git 仓库
 
#解法
[student@workstation ~]$ git clone https://git.lab.example.com/student/create_users_complex.git
[student@workstation create_users_complex]$ vi manage_accounts.yml
---
- name: create user complex
  hosts: dev prod
  become: yes
  vars_files:
    - user_information.yml
  tasks:
    - name: create user us complex envirt
      ansible.builtin.user:
        name: "{{ item.name }}"
        uid: "{{ item.uid }}"
        comment: "{{ item.first | capitalize }} {{ item.middle | capitalize }} {{ item.last | capitalize }}"
        password: "{{ lookup('password', 'password-{{ item.name }} chars=digits length=6 encrypt=sha512_crypt') }}"
      loop: "{{ users }}"
[student@workstation create_users_complex]$ ansible-navigator run -m stdout manage_accounts.yml
[student@workstation create_users_complex]$ cat password-*
[student@workstation create_users_complex]$ ssh servera "tail -3 /etc/passwd"
[student@workstation create_users_complex]$ git add .
[student@workstation create_users_complex]$ git commit -m 'create user complex'
[student@workstation create_users_complex]$ git push
```

## 7. 安装Collection

```bash
安装 hub.lab.example.com 上的 newswangerd.collection_demo Collection，使用 admin 用户， Collection 安装在 /home/student/mycollections 目录

#解法
[student@workstation ~]$ mkdir mycollections
[student@workstation mycollections]$ ansible-galaxy collection install -p /home/student/mycollections ~/newswangerd-collection_demo-1.0.11.tar.gz
[student@workstation mycollections]$ ansible-galaxy collection list -p /home/student/mycollections
```

## 8. 创建自定义Collection

```bash
使用 https://git.lab.example.com/student/custom_collection.git 完成以下要求：
在项目中包含以下资源
    tasks_main.yml
    users.conf
不要对以上文件做任何修改
在控制节点创建自定义 Collection 并满足以下要求：
    Collection 名称叫做 rhel.user
    Collection 包含一个角色叫做 newuser
    文件 tasks_main.yml 存放在 roles/newuser/tasks/main.yml
    文件 users.conf 存放在 roles/newusers/files/users.conf
    上传 Collection rhel.user 到 hub.lab.example.com 的 rhel 命名空间下
（如果 Collection 未显示在 Content Hub， 且已按照正确流程上传，递增 Collection 的版本并再次上传）

# 解法
[student@workstation ~]$ git clone https://git.lab.example.com/student/custom_collection.git
[student@workstation ~]$ cd custom_collection/
[student@workstation custom_collection]$ ansible-galaxy collection init rhel.user

[student@workstation custom_collection]$ ansible-galaxy role init rhel/user/roles/newuser
[student@workstation custom_collection]$ cp tasks_main.yml rhel/user/roles/newuser/tasks/main.yml
[student@workstation custom_collection]$ cp users.conf rhel/user/roles/newuser/files/
[student@workstation custom_collection]$ mkdir rhel/user/meta
[student@workstation custom_collection]$ vi rhel/user/meta/runtime.yml
---
requires_ansible: ">=2.10"
[student@workstation custom_collection]$ ansible-galaxy collection build rhel/user/
[kiosk@foundation0 ~]$ scp student@workstation:custom_collection/rhel-user-1.0.0.tar.gz /home/kiosk/Downloads

#以student用户登录hub.lab.example.com，创建namespace，并将collection上传
```

![image-20241203213358203](https://typora.fanyops.com/documents/2024/1202/image-20241203213358203.png)

## 9. 构建自定义执行环境-1

```bash
根据下列要求构建自定义执行环境：
    自定义执行环境命名为 ee-user-supported:2.2
    执行环境基本镜像为 ee-supported-rhel8:latest
    执行环境的构建镜像为 ansible-builder-rhel8:latest
    自定义执行环境包含 rhel.user 的自定义 Collection
上传新的执行环境到 hub.lab.example.com 中

# 解法
[student@workstation ~]$ mkdir custom_build1
[student@workstation custom_build1]$ vi execution-environment.yml
---
version: 1
build_arg_defaults:
  EE_BASE_IMAGE: 'ee-supported-rhel8:latest'
  EE_BUILDER_IMAGE: 'ansible-builder-rhel8:latest'

dependencies:
  galaxy: requirements.yml
[student@workstation custom_build1]$ vi requirements.yml
---
collections:
  - name: /build/rhel-user-1.0.0.tar.gz
    type: file
[student@workstation custom_build1]$ sudo yum -y install ansible-builder
[student@workstation custom_build1]$ ansible-builder create
[student@workstation custom_build1]$ cp ~/custom_collection/rhel-user-1.0.0.tar.gz ./context/_build/
[student@workstation custom_build1]$ ansible-builder build --tag ee-user-supported:2.2
#由于缺少qemu-img包，会导致构建失败
[student@workstation ~]$ podman load -i ~/ee-user-supported.tar.xz
[student@workstation ~]$ podman images
[student@workstation ~]$ podman tag localhost/ee-user-supported:2.2 hub.lab.example.com/ee-user-supported:2.2
[student@workstation ~]$ podman push hub.lab.example.com/ee-user-supported:2.2
```

## 10. 构建自定义执行环境-2

```bash
按照下列要求创建自定义执行环境：
    执行环境叫做 ee-dyninventory:1.0
    执行环境使用的基本镜像为 ee-supported-rhel8:latest
    执行环境使用的构建镜像为 ansible-builder-rhel8:latest
    执行环境包含 Python36 软件包： python3-ldap 和 python36
上传执行环境到 hub.lab.example.com

# 解法
[student@workstation ~]$ mkdir custom_build2
[student@workstation ~]$ cd custom_build2
[student@workstation custom_build2]$ vi execution-environment.yml
---
version: 1
build_arg_defaults:
  EE_BASE_IMAGE: 'ee-supported-rhel8:latest'
  EE_BUILDER_IMAGE: 'ansible-builder-rhel8:latest'

dependencies:
  system: bindep.txt
[student@workstation custom_build2]$ vi bindep.txt
python3-ldap [platform:rpm]
python36 [platform:rpm]
[student@workstation custom_build2]$ ansible-builder create
[student@workstation custom_build2]$ ansible-builder build --tag ee-dyninventory:1.0
[student@workstation custom_build2]$ podman tag localhost/ee-dyninventory:1.0 hub.lab.example.com/ee-dyninventory:1.0
[student@workstation custom_build2]$ podman push hub.lab.example.com/ee-dyninventory:1.0
```

## 11. 在执行环境中运行剧本

```bash
使用 https://git.lab.example.com/student/dynamic_inventory.git 完成下列要求:
该仓库中包含以下资源:
    ansible 默认配置文件 ansible.cfg
    动态清单脚本 ldap-freeipa.py
注意: ldap-freeipa.py 要求 Python3.6 包:python36 和 python3-ldap
创建一个脚本命名为 main.sh ，用于在执行环境中运行 main.yml 的剧本，满足以下要求:
    使用 ldap-freeipa.py 作为清单
    剧本叫做 main.yml 用于部署/etc/motd.d/banner 文件，内容为 "The Sun comes up and then it goes down"
    剧本 main.yml 用在主机组 testing，除此之外其他主机不应接受该文件
    commit 并 push 变更到 git 仓库

# 解法
[student@workstation ~]$ git clone  https://git.lab.example.com/student/dynamic_inventory.git
[student@workstation ~]$ cd dynamic_inventory/
[student@workstation dynamic_inventory]$ vi main.yml
---
- name: banner
  hosts: testing
  become: yes
  tasks:
    - name: mkdir
      ansible.builtin.file:
        path: /etc/motd.d/
        state: directory
    - name: /etc/motd.d/banner
      ansible.builtin.copy:
        content: 'The Sun comes up and then it goes down'
        dest: /etc/motd.d/banner
[student@workstation dynamic_inventory]$ vi main.sh
#!/bin/bash
ansible-navigator run main.yml -i ldap-freeipa.py --eei hub.lab.example.com/ee-dyninventory:1.0
#在此lab环境执行会失败，这里使用默认的镜像来运行。如果考试也出现此类问题，使用同样的方法。
[student@workstation dynamic_inventory]$ chmod +x main.sh
[student@workstation dynamic_inventory]$ chmod +x ldap-freeipa.py
[student@workstation dynamic_inventory]$ ./main.sh
[student@workstation dynamic_inventory]$ git add .
[student@workstation dynamic_inventory]$ git commit -m dynamic
[student@workstation dynamic_inventory]$ git push
```

## 12. 在剧本中使用变量

```bash
使用 https://git.lab.example.com/student/master_playbook.git 完成以下要求:
仓库内容包含以下资源:
    ansible 默认配置文件 ansible.cfg
    动态清单脚本 inventory.py
创建剧本用于部署文件并且使用下列变量:
    剧本命名为 master_playbook.yml
    剧本运行在主机组 testing 上
    剧本有 3 个变量
        1. content
        2. directory
        3. file
    剧本部署在 directory 目录下的包含有 content 内容的 file 文件
    其他主机不应接受该文件
    commit 并 push 变更到 git 仓库
    
# 解法
[student@workstation ~]$ git clone https://git.lab.example.com/student/master_playbook.git
[student@workstation ~]$ cd master_playbook/
[student@workstation master_playbook]$ chmod +x inventory.py
[student@workstation master_playbook]$ vi master_playbook.yml
---
- name: content directory file
  hosts: testing
  become: yes
  tasks:
    - name: mkdir
      ansible.builtin.file:
        path: "{{ directory }}"
        state: directory
      when: directory is defined
    - name: debug1
      ansible.builtin.debug:
        msg: please define directory variable
      when: directory is not defined
    - name: content directory file task
      ansible.builtin.copy:
        content: "{{ content }}"
        dest: "{{ directory }}/{{ file }}"
      when:
        - directory is defined
        - file is defined
        - content is defined
    - name: debug2
      ansible.builtin.debug:
        msg: please define content variable
      when:
        - directory is defined
        - content is not defined
    - name: debug3
      ansible.builtin.debug:
        msg: please define file variable
      when:
        - directory is defined
        - content is defined
        - file is not defined
[student@workstation master_playbook]$ ansible-playbook -i inventory.py master_playbook.yml -e directory=/tmp/automation -e content='AAP2.2' -e file=info
[student@workstation master_playbook]$ git add --chmod=+x inventory.py
[student@workstation master_playbook]$ git commit -m var
[student@workstation master_playbook]$ git push
[student@workstation master_playbook]$ ssh serverf 'cat /tmp/automation/info'
```

## 13. 创建剧本

```bash
使用 git 仓库 https://git.lab.example.com/student/master_user.git 完成以下内容:
    创建叫做 main.yml 的剧本
    剧本使用 rhel.user Collection
    当剧本运行时，使用 newuser 角色创建用户
    用户仅在清单 tesing 主机组中创建
    commit 并 push 变更到 git 仓库

# 解法
[student@workstation ~]$ git clone https://git.lab.example.com/student/master_user.git
[student@workstation ~]$ cd master_user/
[student@workstation master_user]$ vi inventory
[testing]
serverf.lab.example.com
[student@workstation master_user]$ vi ansible.cfg
[defaults]
collections_paths = /home/student/mycollections
inventory = ./inventory
remote_user = devops
ask_pass = false

[glaxy]
server_list = published_repo

[galaxy_server.published_repo]
url = https://hub.lab.example.com/api/galaxy/content/published/
token = 11ac1b273253a5a5e45883b00aacfff49f3830c4

[privilege_escalation]
become = false
become_method = sudo
become_user = root
become_ask_pass = false
[student@workstation master_user]$ vi main.yml
---
- name: user
  hosts: testing
  become: yes
  roles:
           - role: rhel.user.newuser
[student@workstation master_user]$ ansible-galaxy collection install rhel.user -p /home/student/mycollections/

ansible-playbook main.yml
rm -f inventory
git add .
git commit -m "playbook"
git push
```

## 14. 配置剧本

```bash
创建 Ansible automation controller 项目:
    Name:DO374 copy file project
    Organization: Default
    Credential Type: Git
    URL: https://git.lab.example.com/student/master_playbook
    
    Name: DO374 user project
    Orgnization: Default
    Credential Type: Git
    URL: https://git.lab.example.com/student/master_user
    
[root@controller ~]# tail -1 /etc/tower/settings.py
AWX_TASK_ENV['GIT_SSL_NO_VERIFY'] = 'True'
    
# 访问https://controller.lab.example.com
# 用户名/密码： admin/redhat
```

![image-20241125202355025](https://typora.fanyops.com/documents/2024/1202/image-20241125202355025.png)

![image-20241125202637887](https://typora.fanyops.com/documents/2024/1202/image-20241125202637887.png)

![image-20241125202845023](https://typora.fanyops.com/documents/2024/1202/image-20241125202845023.png)

## 15. 配置清单

```bash
按照下列要求配置 Ansible automation controller 清单：
    DO374 static inventory 包含下列主机组:
    主机组 development 包含 serverd.lab.example.com
    主机组 testing 包含 serverf.lab.example.com
    DO374 dynamic inventory 包含 DO374 custom source，其中包含项目 DO374 copy file
project 中的清单脚本 inventory.py
注意： 在启动前清单源会自动更新
除了上面提到的资源外，不要创建任何其他资源
```

![image-20241125204734531](https://typora.fanyops.com/documents/2024/1202/image-20241125204734531.png)

![image-20241125204912322](https://typora.fanyops.com/documents/2024/1202/image-20241125204912322.png)

![image-20241125205219172](https://typora.fanyops.com/documents/2024/1202/image-20241125205219172.png)

![image-20241125205319911](https://typora.fanyops.com/documents/2024/1202/image-20241125205319911.png)

![image-20241125205427768](https://typora.fanyops.com/documents/2024/1202/image-20241125205427768.png)

![image-20241125205558259](https://typora.fanyops.com/documents/2024/1202/image-20241125205558259.png)

## 16. 配置Ansible执行环境

```bash
在 Ansible automation controller 中使用 hub.lab.example.com/ee-user-supported:2.2执行镜
像创建一个名为 DO374 custom user execution environment 的执行环境
如果在运行前镜像不存在，请先下载
```

![image-20241125210004481](https://typora.fanyops.com/documents/2024/1202/image-20241125210004481.png)

## 17. 配置模板

```bash
按照下列要求创建 Ansible automation controller 作业模板:
    模板 DO374 static copy project template
    当启动模板，则会以 DO374 statck inventory 为清单中主机运行在项目 DO374 copy file project 中的 master_playbook.yml
 设置 DO374 static copy project 模板的变量
---
directory: "/etc/motd.d"
file: "todays_message"
content: "Where there is a will, there is a way"
 模板 DO374 dynamic copy project template
    当启动模板，则以 DO374 dynamic inventory 为清单中主机运行在项目 DO374 copy file project 中的 master_playbook.yml
    设置 DO374 dynamic copy project 模板的变量
---
directory: "/etc/issue.d"
file: "todays_issue"
content: "Experience is the mother of wisdom."
    模板 DO374 user project template
    当启动模板，则以 DO374 static inventory 为清单主机运行在项目 DO374 user project 中的 main.yml 剧本
    模板使用 DO374 custom user execution execution environment 执行环境

# 解法
firefox https://controller.lab.example.com &
Credential-->Add:
Name: host credential
Organizaiton: Default
Credential Type: Machine
Username: student
Password: student
Privilege Escalation Method: sudo
Privilege Escalation Username: root
Privilege Escalation Password: student
```

![image-20241125211754510](https://typora.fanyops.com/documents/2024/1202/image-20241125211754510.png)

![image-20241125211807709](https://typora.fanyops.com/documents/2024/1202/image-20241125211807709.png)

![image-20241125212114172](https://typora.fanyops.com/documents/2024/1202/image-20241125212114172.png)

> ssh root@serverf
>
> 欢迎信息为:The sun goes down, and then the moon comes up

![image-20241125212445177](https://typora.fanyops.com/documents/2024/1202/image-20241125212445177.png)

> ssh root@serverf
>
> cat /etc/issue.d/todays_issue-->内容应为:After the moon goes down, the sun comes up

![image-20241125212759744](https://typora.fanyops.com/documents/2024/1202/image-20241125212759744.png)

