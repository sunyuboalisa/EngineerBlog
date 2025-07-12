---
title: Ansible
draft: false
tags:
---
![[Pasted image 20250612102855.png]]
## 开发IDE：
- VS Code
- python extension pack
- ansible extension pack
- python
- ansible-core
其他的ansible EEs
## Ansible tooling for EEs[](https://docs.ansible.com/ansible/latest/getting_started_ee/introduction.html#ansible-tooling-for-ees "Link to this heading")

Projects in the Ansible ecosystem also provide several tools that you can use with EEs, such as:

- [Ansible Builder](https://ansible-builder.readthedocs.io/en/stable/)
    
- [Ansible Navigator](https://ansible-navigator.readthedocs.io/)
    
- [Ansible AWX](https://ansible.readthedocs.io/projects/awx/en/latest/userguide/execution_environments.html#use-an-execution-environment-in-jobs)
    
- [Ansible Runner](https://ansible-runner.readthedocs.io/en/stable/)
    
- [VS Code Ansible](https://marketplace.visualstudio.com/items?itemName=redhat.ansible)
    
- [Dev Containers extensions](https://code.visualstudio.com/docs/devcontainers/containers)
## 核心概念
### ssh
### Control node
运行ansible CLI tools（ansible-playbook,ansible,ansible-vault）的机器，用来管理网络中的其他机器。
### Managed nodes
被ansible管理的目标机器。可以指定每个节点的信息，ip、用户等。也可以用于分组。
### Inventory
一个提供管理节点列表的清单
默认两个分组是 all 和 ungrouped
创建分组
```
mail.example.com

[webservers]
foo.example.com
bar.example.com

[dbservers]
one.example.com
two.example.com
three.example.com

# yaml格式
ungrouped:
  hosts:
    mail.example.com:
webservers:
  hosts:
    foo.example.com:
    bar.example.com:
dbservers:
  hosts:
    one.example.com:
    two.example.com:
    three.example.com:
```
分组中父子关系
```
# ini 中可以直接:children
[prod:children]
east

# yaml
ungrouped:
  hosts:
    mail.example.com:
webservers:
  hosts:
    foo.example.com:
    bar.example.com:
dbservers:
  hosts:
    one.example.com:
    two.example.com:
    three.example.com:
east:
  hosts:
    foo.example.com:
    one.example.com:
    two.example.com:
west:
  hosts:
    bar.example.com:
    three.example.com:
prod:
  children:
    east:
test:
  children:
    west:
```
主机范围
```
[webservers]
www[01:50].example.com

webservers:
  hosts:
    www[01:50].example.com:
```
分配变量
```
[atlanta]
host1 http_port=80 maxRequestsPerChild=808
host2 http_port=303 maxRequestsPerChild=909

# yaml
atlanta:
  hosts:
    host1:
      http_port: 80
      maxRequestsPerChild: 808
    host2:
      http_port: 303
      maxRequestsPerChild: 909


[targets]

localhost              ansible_connection=local
other1.example.com     ansible_connection=ssh        ansible_user=myuser
other2.example.com     ansible_connection=ssh        ansible_user=myotheruser
```

定义变量给给分组
```
[atlanta]
host1
host2

[atlanta:vars]
ntp_server=ntp.atlanta.example.com
proxy=proxy.atlanta.example.com

# yaml
atlanta:
  hosts:
    host1:
    host2:
  vars:
    ntp_server: ntp.atlanta.example.com
    proxy: proxy.atlanta.example.com
```
 管理变量
 在附带的默认vars插件下，可以用host和group变量文件。可以是yml、yaml、json等类型。会漠然加载一些路径下的变量文件：
```
 # 常用的一种结构
 inventory/
├── hosts
├── group_vars/
│   └── webservers.yml
└── host_vars/
    └── web1.yml

```
加载也是有优先级的：
1. command line values (for example, , these are not variables)`-u my_user`
    
2. role defaults (as defined in [Role directory structure](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_reuse_roles.html#role-directory-structure)) [1](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_variables.html#id14)
    
3. inventory file or script group vars [2](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_variables.html#id15)
    
4. inventory group_vars/all [3](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_variables.html#id16)
    
5. playbook group_vars/all [3](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_variables.html#id16)
    
6. inventory group_vars/* [3](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_variables.html#id16)
    
7. playbook group_vars/* [3](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_variables.html#id16)
    
8. inventory file or script host vars [2](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_variables.html#id15)
    
9. inventory host_vars/* [3](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_variables.html#id16)
    
10. playbook host_vars/* [3](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_variables.html#id16)
    
11. host facts / cached set_facts [4](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_variables.html#id17)
    
12. play vars
    
13. play vars_prompt
    
14. play vars_files
    
15. role vars (as defined in [Role directory structure](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_reuse_roles.html#role-directory-structure))
    
16. block vars (only for tasks in block)
    
17. task vars (only for the task)
    
18. include_vars
    
19. set_facts / registered vars
    
20. role (and include_role) params
    
21. include params
    
22. extra vars (for example, )(always win precedence)`-e "user=my_user"`


### Ansible CLIs
Ad Hoc
适合执行很少重复的任务
```
ansible [pattern] -m [module] -a "[module options]"
```
Ansible-Playbook

### Playbooks
- Playbook
A list of plays that define the order in which Ansible performs operations, from top to bottom, to achieve an overall goal.
- Play
The main context for Ansible execution, this playbook object maps managed nodes (hosts) to tasks. The Play contains variables, roles and an ordered lists of tasks and can be run repeatedly. It basically consists of an implicit loop over the mapped hosts and tasks and defines how to iterate over them.
- Task
应用与Manage Node上的 action
- Roles
A limited distribution of reusable Ansible content (tasks, handlers, variables, plugins, templates and files) for use inside of a Play.
To use any Role resource, the Role itself must be imported into the Play
- Handlers
一种特殊的Task，仅在前一个结果是changed状态的task通知时执行

#### Templating(Jinja2)
在目标机器上发送和执行任务**之前**，所有模板都在 Ansible 控制节点上。 这种方法最大限度地减少了目标上的包要求（jinja2 仅在控制节点上需要）。 它还限制了 Ansible 传递到目标机器的数据量。 Ansible 解析控制节点上的模板，并仅将每个任务所需的信息传递给目标机器，而不是在控制节点上传递所有数据并在目标上解析。
```
├── hostname.yml
├── templates
    └── test.j2
```
hostname.yml
```
---
- name: Write hostname
  hosts: all
  tasks:
  - name: write hostname using jinja2
    ansible.builtin.template:
       src: templates/test.j2
       dest: /tmp/hostname
```
### Modules
Ansible 复制到每个托管式节点并在每个托管式节点上执行的代码或二进制文件（需要时），以完成每个 Task 中定义的作。
### Plugins
扩展ansible的功能。 [Working with plugins](https://docs.ansible.com/ansible/latest/plugins/plugins.html#working-with-plugins)
### Collections
一种分发 Ansible 内容的格式，可以包含 playbook、角色、模块和插件。可以通过Ansible Galaxy来安装和使用Collections。  [Collection Index](https://docs.ansible.com/ansible/latest/collections/index.html#list-of-collections)， [Using Ansible collections](https://docs.ansible.com/ansible/latest/collections_guide/index.html#collections).
## 教程
Hello World入门
0.准备ssh环境
```

```
1.创建inventory文件，可以用ini或者yaml格式的
```
# ini 格式
[myhosts]
192.0.2.50 ansible_user=root
192.0.2.51 ansible_user=root
192.0.2.52 ansible_user=root

# yaml 格式
myhosts:
  hosts:
    my_host_01:
      ansible_host: 192.0.2.50 ansible_user: root
    my_host_02:
      ansible_host: 192.0.2.51 ansible_user: root
    my_host_03:
      ansible_host: 192.0.2.52 ansible_user: root

```
2.验证inventory
```
# 假设文件名是 inventory.ini
ansible-inventory -i inventory.ini --list
# 假设文件名是 inventory.yaml
ansible-inventory -i inventory.yaml --list
```
3.创建playbook
```
---
- name: My first play
  hosts: myhosts
  tasks:
   - name: Ping my hosts
     ansible.builtin.ping:

   - name: Print message
     ansible.builtin.debug:
       msg: Hello world
```
4.用ansible执行
```
ansible-playbook -i inventory.int playbook.yaml
```
工具


参考：
1.https://docs.ansible.com/ansible/latest/getting_started/get_started_inventory.html