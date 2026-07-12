---
title: NTTU 伺服器原理 WriteUp-2
date: 2024-08-09 15:49:07
tags: 
  - server
  - csie
category: csie
---

## Ansible

### Docs

+ `$ansible-navigator doc -l`
+ `ansible-navigator doc <ansible.builtin.dnf>`

[Official](https://docs.ansible.com)

![image.png](https://hackmd.io/_uploads/By-XF2wX6.png)

### Yaml

[Re](/4KwXbbiGQd2tMdJVML-zxA)

![image.png](https://hackmd.io/_uploads/H1MBF2vQT.png)

### Basic

#### 名詞解釋

+ inventory
    + host
    + group
+ playbook
    + play
    + task
        + modules

#### Requirement
Linux , macos ,unix like host: **need >python3.5**

#### Inventory

+ 定義清單

```bash=
$cd /etc/ansible
$vim hosts
```

> INI or YAML

+ 群組

```plaintext
[bind_server]
servera.lab.example.com
serverb.lab.example.com
(server count)>=0
[web_server]
.....
```

+ 巢狀群組

```vim
[usa]
washington1.example.com
washington2.example.com
[canada]
ontario01.example.com
ontario02.example.com
[north-america:children]
canada
usa
```

#### ansible.cfg

+ `$vim ansible.cfg`

```vim=
[defaults]
inventory = ./inventory
remote user = devops
ask_pass = false
# $ssh-keygen
# $ssh-copy-id
[privilege_escalation]
become = true
become method = sudo
become_user = root
become_ask_pass = true
```

+ Show inventory

```bash=
$cp /etc/ansible/hosts inventory
$ansible-navigator inventory -m stdout <--list or --graph>
```

#### Playbook.yml

```yml=
---
- name: Adduser
  hosts: bind_server
  tasks:
    - name: username is chih 
      ansible.builtin.user:
        name: chih 
        uid: 8888
        state: present
```

#### Run

```bash
$ansible-navigator run -m stdout playbook.yml
```

### Ansible simple buildup

#### Ansible.cfg & Inventory

+ `$vim ansible.cfg`

```vim=
[defaults]
inventory = ./inventory
remote_user = devops
ask_pass = false
# $ssh-keygen
# $ssh-copy-id
[privilege_escalation]
become = true
become_method = sudo
become_user = root
become_ask_pass = false
```

+ `vim inventory`

```vim=
[bind_server]
servera.lab.example.com
serverb.lab.example.com

[unbound_server]
servverc.lab.example.com
```

+ `$ansible-navigator inventory --graph -m stdout`
+ `$ansible-navigator exec --ansible -m ansible.builtin.user -a 'name=rcwang' bind_server`
    + ansibel.builtin.user -a : bliud user
    + bind_server : In bind_server

#### Playbook

+ `$vim newuser.yml`

```yml=
---
- name: useradd
  hosts: bind_server
  tasks:
    - name: adduser jjli
      ansible.builtin.user:
        name: jjli
        state: present  # default
        
    - name: adduser hcyang
      ansible.builtin.user:
        name: hcyang
...
```

+ `$vim bind_server.yml`

```yml=
---
# This file is setup by <name> at <time> for setting up <server>
#
#
- name: bind-chroot server setup
  hosts: bind_server
  tasks:
    - name: Step1. Installation
      ansible.builtin.package:
        name: bind-chroot
        state: present
    
    - name: Step2. First Start
      ansible.builtin.service:
        name: named-chroot
        enable: true
```


#### Other parameter

+ 增加輸出的細部資訊
    + `$ansibel-navigator run newuser.yml -m stdout -vvvv`

| 選項v  | 描述|
| ----- | -------------------------------------------------------------------------------------------- |
| -v    | 顯示任務結果。                                                                               |
| -vv   | 任務結果和任務配置都會顯示                                                                   |
| -vvv  | 包含關於與受管主機連接的資訊                                                                 |
| -vvvv | 增加了連接外掛程式相關的額外詳細程度選項，包括受管主機上用於執行腳本的用戶，以及所執行的腳本 |

+ `$ansible-navigator run newuser.yml -m stdout --syntax-check`
    + 檢查語法錯誤 or 縮排

### Variable & Fact

#### Playbook variable

+ `$vim newuser.yml`
```yml=
---
- name: useradd
  hosts: bind_server
  vars: 
    user1: chiawei
    uid1: 5566
  vars_files:
    - userlist.yml #
  tasks:
    - name: adduser "{{user1}}"
      ansible.builtin.user:
        name: "{{user1}}"
        uid: "{{uid1}}"
        state: present  # default
        
    - name: adduser "{{ user2 }}"
      ansible.builtin.user:
        name: "{{ user2 }}"
        uid: "{{uid2}}"
...
```
+ `$vim userlist.yml`
```yml=
user2: fan
uid2: 1234
```

#### Host & group variable

+ `vim inventory`

```plaintext=
[bind_server]
servera.lab.example.com ip4.addr=172.25.250.11 # Host variable
serverb.lab.example.com ip4.addr=172.25.250.12

[bind_server:vars] # group variable
install_sodtware = bind-chroot
service_name = named-chroot

[unbound_server]
servverc.lab.example.com
```

+ `$mkdir group_vars host_vars`

```bash=
$mkdir group_vars 
$mkdir host_vars
$cd group_vars
$vim bind_server
$cd ..
$cd host_vars
$vim servera.lab.example.com
$vim serverb.lab.example.com
```

+ `$vim bind_server`

```text=
install_software = bind-chroot
service_name = named-chroot
```

+ `$vim servera.lab.example.com`

```text=
ipv4_addr = 172.25.250.11
```
+ `$vim serverb.lab.example.com`

```text=
ipv4_addr = 172.25.250.12
```

+ `$tree`

![image.png](https://hackmd.io/_uploads/HkTGL0wmT.png)


#### Register
+ register 語法可以取得命令輸出，並保存在一個臨時變數中

+ `$vim newuser.yml`
```yml=
---
- name: Addusers
  hosts: all
  tasks:
    - name: add different user in different server
      ansible.builtin.user:
        name: "{{newuser_name}}"
      register: useradd_result
    - name: Print results
      ansible.builtin.debugg:
        var: useradd_result
...
```

+ `$ansible-navigator run newuser.yml -m stdout`

![image.png](https://hackmd.io/_uploads/S1Eos0wXT.png)

#### Vault




### Task control

+ `$vim user_add.yml`
```yml=
- name: User Management
  hosts: serverd.lab.example.com
  vars:
    user_lists:
      - name: hcyang 
        user_id: 1236
      - name: jjli 
        user_id: 1235
  tasks:
    - name: adduser {{ item.name }} 
      ansible.builtin.user:
        name: "{{ item.name }}" 
        uid: "{{ item.user_id }}"
        state: present
      loop: "{{ user_lists }}" 
      register: useradd_result
    - name: print results 
      ansible.builtin.debug:
        msg: |
        An user is add with name {{ item.name }} 
      loop: "{{ useradd_result. results }}"
```

## 期末考整理

`DNS Http varnish database`

[DNS](https://www.youtube.com/watch?v=VhXenrYuYcE)

[HTTP_server](https://www.youtube.com/watch?v=0em4LrNN_v8)

[Https cerify ~1:10:00](https://www.youtube.com/watch?v=q14Aunuv9WU)

[Varnish and haproxy 1:10:00 ~ ](https://www.youtube.com/watch?v=q14Aunuv9WU)

[Database]()
