<h1 align="center">Easy-in-use and cost-efficient solution for SSH management</h1>

![](Ansible-ssh.jpg)


## SSH
The **SSH** protocol is the global gold standard for remote system administration and secure file transfer. SSH (Secure Shell) is used in every data center and in every major enterprise. One of the features behind the immense popularity of the protocol is the strong authentication using SSH keys.

[Documantation](https://www.openssh.com/manual.html)
## Ansible
**Ansible** is a radically simple IT automation engine that automates cloud provisioning, configuration management, application deployment, intra-service orchestration, and many other IT needs.

[Documantation](https://www.ansible.com)

[Example of Ansible configuration](https://github.com/codica2/ansible-best-practice)
## How to use

### Prerequisites

**You will need:**

1. Remote host IP.
2. Keys in .yaml extension.

**Steps to configure playbook**

1. Add your IPs to [hosts file](https://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html).
2. Add your host pattern into 'hosts' field.
3. Enter Username into 'remote_user' field.
4. Enter project name into 'project' field.
5. Specify path to your .ssh/authorized_keys.
6. OPTIONALLY. By default, file authorized_keys will be replaced by new one. If you need to append your keys to file, you need to change ```tr -d "[']" >" to "tr -d "[']" >>```
7. Shell command to ping your nodes: 'ansible -i hosts test -m ping'. # test is your hosts sub-pattern specified in hosts file [test],[aws] etc.
8. Shell command to execute playbook: 'ansible-playbook -i hosts change-keys.yaml'.


### Storing SSH keys
We store keys in [user.yaml](users.yaml) file, where each user has own SSH key and list of projects he will need access to. 

```yaml
ssh:
  - name: user_1
    key: ["ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AA/ZUuJP7XwDphF+ssgdQzIgXg5 user1.codica@gmail.com"]
    projects: ["project1","project2"]
  - name : user_2
    key: ["ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AA/ZUuJP7XwDphF+ssgdQzIgXg5 user2.codica@gmail.com"]
    projects: ["project1","project3"]
  - name: user_3
    key: ["ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AA/ZUuJP7XwDphF+ssgdQzIgXg5 user3.codica@gmail.com"]
    projects: ["project3"]

```
### Playbook for adding SSH keys
Our playbook will add keys of needed users to remote servers, it will rewrite existing `.ssh/authorized_keys` file, that's why we can be sure, that we store only authorized keys

```yaml
---
- hosts: # Set host or a pattern specified in hosts file. You can input address pool here like: [172.17.0.2]. Specify port if :22 is blocked
  remote_user: user_name  # Choosing user to become on remote machine
  become: true
  vars:
    project: ["project_name"] # Enter the project name to add to the remote host
 
  tasks:
  - name: Including keys as variables
    include_vars:
      file: users.yaml # Choosing file with keys 

  - blockinfile:
      marker: "#{{item.name}}"
      block: "{{item.key}}"
      create: yes
      dest: /home/.tmpkeys # Temporary file is needed to format keys into authorized_keys
      state: present
    when: project is subset (item.projects) 
    with_items: "{{ ssh }}"
  - shell: cat /home/.tmpkeys | tr -d "[']" | tr ',' '\n' > /home/dev/.ssh/authorized_keys && rm /home/.tmpkeys  # Specify path to .ssh/authorized_keys

```

After running this playbook, we will have `.ssh/authorized_keys` with following format:

```
#user_1
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AA/ZUuJP7XwDphF+ssgdQzIgXg5 user1.codica@gmail.com
#user_1
#user_2
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AA/ZUuJP7XwDphF+ssgdQzIgXg5 user2.codica@gmail.com
#user_2
```

## License
Copyright © 2015-2019 Codica. It is released under the [MIT License](https://opensource.org/licenses/MIT).

## About Codica

[![Codica logo](https://www.codica.com/assets/images/logo/logo.svg)](https://www.codica.com)

The names and logos for Codica are trademarks of Codica.

We love open source software! See [our other projects](https://github.com/codica2) or [hire us](https://www.codica.com/) to design, develop, and grow your product.