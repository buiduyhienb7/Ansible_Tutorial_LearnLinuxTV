# Ansible tutorial
### 1. Introdution
Ansible is an incredible configuration management and provisioning utility that enables you to automate all the things. In this series, you’ll learn everything you need to know in order to use Ansible for your day-to-day administration duties. In the first video in this series, we take a look at the overall theme and what to expect.
### 2. SSH Overview & setup
- Generate an ssh key: `ssh-keygen -t ed25519 -C "jay default"`
- Copy the ssh key to the server: `ssh-copy-id -i ~/.ssh/id_ed25519.pub 192.168.10.51`
- Generate an ssh key that’s going to be specifically used for Ansible: `ssh-keygen -t ed25519 -C "ansible"`
- Copy the ssh key to the servers: `ssh-copy-id -i ~/.ssh/ansible.pub 192.168.10.51`
- Use an ssh key to connect to a server: `ssh -i ~/.ssh/<keyname> <ip address>`
- To cache the passphrase for our session, we can use the ssh agent: `eval $(ssh-agent) && ssh-add`
- Here’s an alias you can put in your .bashrc, to simplify it: `vi .bashrc`
```bash
alias ssha='eval $(ssh-agent) && ssh-add'
```
### 3. Setting up the git repo
- Check if git is installed: `which git`
- Install git: `sudo apt update` `sudo apt install git`
- Create user name config for git: `git config --global user.name "buiduyhienb7"`
- Create user email config for git: `git config --global user.email "somebody@somewhere.net"`
- Check the status of your git repository: `git status`
- Stage the README.md file (after making changes) to be included in the next git commit: `git add README.md`
- Set up the README.md file to be included in a commit: `git commit -m "Updated readme file, initial commit"`
- Send the commit to Github: `git push origin master`
### 4. Running ad-hoc commands
- List all of the hosts in the inventory `ansible all --list-hosts`
- Gather facts about your hosts `ansible all -m gather_facts`
- Gather facts about your hosts, but limit it to just one host: `ansible all -m gather_facts --limit 172.16.250.132`
### 5. Running elavated ad-hoc commands
### 6. Writing our first playbook
- Writing first playbook install_apache.yml: `vi install_apache.yml`
```yml
--- 

- hosts: all 
  become: true 
  tasks:

  - name: install apache2 package 
    apt: 
      name: apache2
```
- Run the playbook: `ansible-playbook --ask-become-pass install_apache.yml`
- Writing playbook install_apache.yml:
```yml
---

- hosts: all
  become: true
  tasks:

  - name: update repository index
    apt:
      update_cache: yes

  - name: install apache2 package
    apt:
      name: apache2
      state: latest
  
  - name: install php support for apache
    apt:
      name: libapache2-mod-php
      state: latest
```
- Writing playbook remove_apache.yml
```yml
---

- hosts: all
  become: true
  tasks:

  - name: install apache2 package
    apt:
      name: apache2
      state: absent
  
  - name: install php support for apache
    apt:
      name: libapache2-mod-php
      state: absent
```
### 7. The "when conditional"
- Gather facts, while limiting to a single host: `ansible all -m gather_facts --limit 192.168.10.54`
- install_apache.yml (updated include AlmaLinux)
```yaml
---

- hosts: all
  become: yes
  tasks:

  - name: update ubuntu's repository
    apt:
      update_cache: yes
    when: ansible_distribution == "Ubuntu"
  
  - name: install apache on Ubuntu
    apt:
      name: apache2
      state: latest
    when: ansible_distribution == "Ubuntu"
  
  - name: install php on Ubuntu
    apt:
      name: libapache2-mod-php
      state: latest
    when: ansible_distribution == "Ubuntu"

  - name: update AlmaLinux repository
    dnf:
      update_cache: yes
    when: ansible_distribution == "AlmaLinux"

  - name: install apache on AlmaLinux
    dnf:
      name: httpd
      state: latest
    when: ansible_distribution == "AlmaLinux"
  
  - name: install php on AlmaLinux
    dnf:
      name: php
      state: latest
    when: ansible_distribution == "AlmaLinux"
```
### 8. Improving your playbook
- in this part, we look into a few ways we can clean up and consolidate the playbook we've been working with so far.
- install_apache.yaml (condensed)
```yaml
- hosts: all
  become: yes
  task:

  - name: update repository on Ubuntu
    apt:
      update_cache: yes
    when: ansible_distribution == "Ubuntu"

  - name: install apache and php packages on Ubuntu
    apt:
      name:
      - apache2
      - libapache2-mod-php
      state: latest
    when: ansible_distribution == "Ubuntu"

  - name: update repository on AlmaLinux
    dnf:
      update_cache: yes
    when: ansible_distribution == "Ubuntu"

  - name: install apache and php packages on AlmaLinux
    dnf:
      name:
      - httpdS
      - php
      state: latest
    when: ansible_distribution == "AlmaLinux"
```
- install_apache.yaml (condensed enven futher), add avairible in inventory file
```yaml
---
 
 - hosts: all
   become: true
   tasks:
 
   - name: install apache and php
     package:
       name:
         - "{{ apache_package }}"
         - "{{ php_package }}"
       state: latest
       update_cache: yes
```
- inventory file
```
192.168.10.51 apache_package=apache2 php_package=libapache2-mod-php
192.168.10.52 apache_package=apache2 php_package=libapache2-mod-php
192.168.10.53 apache_package=apache2 php_package=libapache2-mod-php
192.168.10.54 apache_package=httpd php_package=php
```
### 9. Targeting Specific Nodes

In this part, we split our inventory file into groups, and look at how to run tasks on nodes based on their group.

- Edit inventory file (updated with groups):

```yaml
[web_servers]
172.16.250.51
172.16.250.54
 
[db_servers]
172.16.250.52

[file_servers]
172.16.250.53
```

- Edit site.yaml (targeting specific nodes)

```yaml
---

- hosts: all
  become: yes
  pre_tasks:

  - name: install updates (Ubuntu)
    apt:
      update_cache: yes
      upgrade: dist
    when: ansible_distribution == "Ubuntu"

  - name: install updates (AlmaLinux)
    dnf:
      update_cache: yes
      update_only: yes
    when: ansible_distribution == "AlmaLinux"


- hosts: web_servers
  become: yes
  tasks:

  - name: install apache and php on Ubuntu
    apt:
      name:
      - apache2
      - libapache2-mod-php
      state: latest
    when: ansible_distribution == "Ubuntu"

  - name: install apache and php on AlmaLinux
    dnf:
      name:
      - httpd
      - php
      state: latest
    when: ansible_distribution == "AlmaLinux"


- hosts: db_servers
  become: yes
  tasks:

  - name: install maria db on Ubuntu
    apt:
      name: mariadb-server
      state: latest
    when: ansible_distribution == "Ubuntu"

  - name: install maria db on AlmaLinux
    dnf:
      name: mariadb
      state: latest
    when: ansible_distribution == "AlmaLinux"


- hosts: file_servers
  become: yes
  tasks:

  - name: install samba on file servers
    package:
      name: samba
      state: latest
```

### 10. Tags

In this part, we learn how to add tags to our plays that can make it easier to target specific things when we dont't want to run the entire playbook each time.

- site_with_tags.yaml
  
```yaml

- hosts: all
  become: yes
  pre_tasks:

  - name: install updates (Ubuntu)
    tags: always
    apt:
      update_cache: yes
      upgrade: dist
    when: ansible_distribution == "Ubuntu"

  - name: install updates (AlmaLinux)
    tags: always
    dnf:
      update_cache: yes
      update_only: yes
    when: ansible_distribution == "AlmaLinux"


- hosts: web_servers
  become: yes
  tasks:

  - name: install apache and php on Ubuntu
    tags: apache,apache2,web,ubuntu
    apt:
      name:
      - apache2
      - libapache2-mod-php
      state: latest
    when: ansible_distribution == "Ubuntu"

  - name: install apache and php on AlmaLinux
    tags: apache,httpd,web,alma
    dnf:
      name:
      - httpd
      - php
      state: latest
    when: ansible_distribution == "AlmaLinux"
  

- hosts: db_servers
  become: yes
  tasks:

  - name: install maria db on Ubuntu
    tags: mariadb_packet,mariadb-server,db,ubuntu
    apt:
      name: mariadb-server
      state: latest
    when: ansible_distribution == "Ubuntu"

  - name: install maria db on AlmaLinux
    tags: mariadb_packet,mariadb,db,alma
    dnf:
      name: mariadb
      state: latest
    when: ansible_distribution == "AlmaLinux"


- hosts: file_servers
  become: yes
  tasks:

  - name: install samba on Ubuntu
    tags: samba,file,ubuntu
    apt:
      name: samba
      state: latest
    when: ansible_distribution == "Ubuntu"

  - name: install samba on Alma
    tags: samba,file,alma
    dnf:
      name: samba
      state: latest
    when: ansible_distribution == "AlmaLinux"
```

- List the available tags in a playbook:`ansible-playbook --list-tags site_with_tags.yaml`
- Examples of running a playbook bot targeting specifiec tags: `ansible-playbook --tags db --ask-become-pass site_with_tags.yml`

### 11. Managing files

- Add files/default_site.html

```html
<html>
     <title>Web-site test</title>
     <body>
        Ansible is awesome!
    </body>
 </html>
```
- Add workstation ip on inventory file

```yaml
[workstation]
192.168.10.50
```
- Copy file default_site.html to sites and install terraform on workstation

```yaml
 ---
 
- hosts: all
  become: yes
  pre_tasks:

  - name: install updates (Ubuntu)
    tags: always
    apt:
      update_cache: yes
      upgrade: dist
    when: ansible_distribution == "Ubuntu"

  - name: install updates (AlmaLinux)
    tags: always
    dnf:
      update_cache: yes
      update_only: yes
    when: ansible_distribution == "AlmaLinux"


- hosts: workstations
  become: yes
  tasks:

  - name: install unzip
    package:
      name: unzip
      state: latest

  - name: install terraform
    unarchive:
      src: https://releases.hashicorp.com/terraform/0.12.28/terraform_0.12.28_linux_amd64.zip
      dest: /usr/local/bin
      remote_src: yes
      owner: root
      group: root
      mode: 0755

- hosts: web_servers
  become: yes
  tasks:

  - name: install apache and php on Ubuntu
    tags: apache,apache2,web,ubuntu
    apt:
      name:
      - apache2
      - libapache2-mod-php
      state: latest
    when: ansible_distribution == "Ubuntu"

  - name: install apache and php on AlmaLinux
    tags: apache,httpd,web,alma
    dnf:
      name:
      - httpd
      - php
      state: latest
    when: ansible_distribution == "AlmaLinux"

  - name: copy default html file for site
    tags: apache,apache2,httpd
    copy:
      src: default_site.html
      dest: /var/www/html/index.html
      owner: root
      group: root
      mode: 0644


- hosts: db_servers
  become: yes
  tasks:

  - name: install maria db on Ubuntu
    tags: mariadb_packet,mariadb-server,db,ubuntu
    apt:
      name: mariadb-server
      state: latest
    when: ansible_distribution == "Ubuntu"

  - name: install maria db on AlmaLinux
    tags: mariadb_packet,mariadb,db,alma
    dnf:
      name: mariadb
      state: latest
    when: ansible_distribution == "AlmaLinux"


- hosts: file_servers
  become: yes
  tasks:

  - name: install samba on Ubuntu
    tags: samba,file,ubuntu
    apt:
      name: samba
      state: latest
    when: ansible_distribution == "Ubuntu"

  - name: install samba on Alma
    tags: samba,file,alma
    dnf:
      name: samba
      state: latest
    when: ansible_distribution == "AlmaLinux"

```

- Run the updated playbook: `ansible-playbook --ask-become-pass file_management.yaml`

### 12. Managing service

- enable http service on AlmaLinux

```yaml
---

- hosts: all
  become: yes
  pre_tasks:

  - name: install updates (Ubuntu)
    tags: always
    apt:
      update_cache: yes
      upgrade: dist
    when: ansible_distribution == "Ubuntu"

  - name: install updates (AlmaLinux)
    tags: always
    dnf:
      update_cache: yes
      update_only: yes
    when: ansible_distribution == "AlmaLinux"


- hosts: workstations
  become: yes
  tasks:

  - name: install unzip
    package:
      name: unzip
      state: latest

  - name: install terraform
    unarchive:
      src: https://releases.hashicorp.com/terraform/0.12.28/terraform_0.12.28_linux_amd64.zip
      dest: /usr/local/bin
      remote_src: yes
      owner: root
      group: root
      mode: 0755


- hosts: web_servers
  become: yes
  tasks:

  - name: install apache and php on Ubuntu
    tags: apache,apache2,web,ubuntu
    apt:
      name:
      - apache2
      - libapache2-mod-php
      state: latest
    when: ansible_distribution == "Ubuntu"

  - name: install apache and php on AlmaLinux
    tags: apache,httpd,web,alma
    dnf:
      name:
      - httpd
      - php
      state: latest
    when: ansible_distribution == "AlmaLinux"

  - name: start and enable httpd service (AlmaLinux)
    tags: apache,httpd,alma
    service:
      name: httpd
      state: started
      enabled: yes
    when: ansible_distribution == "AlmaLinux"

  - name: copy default html file for site
    tags: apache,apache2,httpd
    copy:
      src: default_site.html
      dest: /var/www/html/index.html
      owner: root
      group: root
      mode: 0644


- hosts: db_servers
  become: yes
  tasks:

  - name: install maria db on Ubuntu
    tags: mariadb_packet,mariadb-server,db,ubuntu
    apt:
      name: mariadb-server
      state: latest
    when: ansible_distribution == "Ubuntu"

  - name: install maria db on AlmaLinux
    tags: mariadb_packet,mariadb,db,alma
    dnf:
      name: mariadb
      state: latest
    when: ansible_distribution == "AlmaLinux"
    

- hosts: file_servers
  become: yes
  tasks:

  - name: install samba on Ubuntu
    tags: samba,file,ubuntu
    apt:
      name: samba
      state: latest
    when: ansible_distribution == "Ubuntu"

  - name: install samba on Alma
    tags: samba,file,alma
    dnf:
      name: samba
      state: latest
    when: ansible_distribution == "AlmaLinux"
```

- change e-mail address for admin
```yaml
---

- hosts: all
  become: yes
  pre_tasks:

  - name: install updates (Ubuntu)
    tags: always
    apt:
      update_cache: yes
      upgrade: dist
    when: ansible_distribution == "Ubuntu"

  - name: install updates (AlmaLinux)
    tags: always
    dnf:
      update_cache: yes
      update_only: yes
    when: ansible_distribution == "AlmaLinux"


- hosts: workstations
  become: yes
  tasks:

  - name: install unzip
    package:
      name: unzip
      state: latest

  - name: install terraform
    unarchive:
      src: https://releases.hashicorp.com/terraform/0.12.28/terraform_0.12.28_linux_amd64.zip
      dest: /usr/local/bin
      remote_src: yes
      owner: root
      group: root
      mode: 0755


- hosts: web_servers
  become: yes
  tasks:

  - name: install apache and php on Ubuntu
    tags: apache,apache2,web,ubuntu
    apt:
      name:
      - apache2
      - libapache2-mod-php
      state: latest
    when: ansible_distribution == "Ubuntu"

  - name: install apache and php on AlmaLinux
    tags: apache,httpd,web,alma
    dnf:
      name:
      - httpd
      - php
      state: latest
    when: ansible_distribution == "AlmaLinux"

  - name: start and enable httpd service (AlmaLinux)
    tags: apache,httpd,alma
    service:
      name: httpd
      state: started
      enabled: yes
    when: ansible_distribution == "AlmaLinux"

  - name: copy default html file for site
    tags: apache,apache2,httpd
    copy:
      src: default_site.html
      dest: /var/www/html/index.html
      owner: root
      group: root
      mode: 0644

  - name: change e-mail address for admin
     tags: apache,alma,httpd
     lineinfile:
       path: /etc/httpd/conf/httpd.conf
       regexp: '^ServerAdmin'
       line: ServerAdmin somebody@somewhere.net
     when: ansible_distribution == "AlmaLinux"
     register: httpd
 
  - name: restart httpd (AlmaLinux)
      tags: apache,alma,httpd
      service:
        name: httpd
        state: restarted
      when: httpd.changed 


- hosts: db_servers
  become: yes
  tasks:

  - name: install maria db on Ubuntu
    tags: mariadb_packet,mariadb-server,db,ubuntu
    apt:
      name: mariadb-server
      state: latest
    when: ansible_distribution == "Ubuntu"

  - name: install maria db on AlmaLinux
    tags: mariadb_packet,mariadb,db,alma
    dnf:
      name: mariadb
      state: latest
    when: ansible_distribution == "AlmaLinux"
    

- hosts: file_servers
  become: yes
  tasks:

  - name: install samba on Ubuntu
    tags: samba,file,ubuntu
    apt:
      name: samba
      state: latest
    when: ansible_distribution == "Ubuntu"

  - name: install samba on Alma
    tags: samba,file,alma
    dnf:
      name: samba
      state: latest
    when: ansible_distribution == "AlmaLinux"
```

### 13. Adding Users & Bootstrapping

- Add section for creating user simone, add ssh key for user to control ansible and add sudoers file for user

```yaml
---

- hosts: all
  become: yes
  pre_tasks:

  - name: install updates (Ubuntu)
    tags: always
    apt:
      update_cache: yes
      upgrade: dist
    when: ansible_distribution == "Ubuntu"

  - name: install updates (AlmaLinux)
    tags: always
    dnf:
      update_cache: yes
      update_only: yes
    when: ansible_distribution == "AlmaLinux"


- hosts: all
  become: yes
  tasks:

  - name: create simone user
    tags: always
    user:
      name: simone
      groups: root

  - name: add ssh key for simone
    tags: always
    authorized_key:
      user: simone
      key: "ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAILInjsYJAQk+CyEOiPmklZltW4kDeTBuy6DWd5IvWloG ansible"

  - name: add sudoers file for simone
    copy:
      src: sudoer_simone
      dest: /etc/sudoers.d/simone
      owner: root
      group: root
      mode: 0440


- hosts: workstations
  become: yes
  tasks:

  - name: install unzip
    package:
      name: unzip
      state: latest

  - name: install terraform
    unarchive:
      src: https://releases.hashicorp.com/terraform/0.12.28/terraform_0.12.28_linux_amd64.zip
      dest: /usr/local/bin
      remote_src: yes
      owner: root
      group: root
      mode: 0755


- hosts: web_servers
  become: yes
  tasks:

  - name: install apache and php on Ubuntu
    tags: apache,apache2,web,ubuntu
    apt:
      name:
      - apache2
      - libapache2-mod-php
      state: latest
    when: ansible_distribution == "Ubuntu"

  - name: install apache and php on AlmaLinux
    tags: apache,httpd,web,alma
    dnf:
      name:
      - httpd
      - php
      state: latest
    when: ansible_distribution == "AlmaLinux"

  - name: start and enable httpd service (AlmaLinux)
    tags: apache,httpd,alma
    service:
      name: httpd
      state: started
      enabled: yes
    when: ansible_distribution == "AlmaLinux"

  - name: copy default html file for site
    tags: apache,apache2,httpd
    copy:
      src: default_site.html
      dest: /var/www/html/index.html
      owner: root
      group: root
      mode: 0644


- hosts: db_servers
  become: yes
  tasks:

  - name: install maria db on Ubuntu
    tags: mariadb_packet,mariadb-server,db,ubuntu
    apt:
      name: mariadb-server
      state: latest
    when: ansible_distribution == "Ubuntu"

  - name: install maria db on AlmaLinux
    tags: mariadb_packet,mariadb,db,alma
    dnf:
      name: mariadb
      state: latest
    when: ansible_distribution == "AlmaLinux"


- hosts: file_servers
  become: yes
  tasks:

  - name: install samba on Ubuntu
    tags: samba,file,ubuntu
    apt:
      name: samba
      state: latest
    when: ansible_distribution == "Ubuntu"

  - name: install samba on Alma
    tags: samba,file,alma
    dnf:
      name: samba
      state: latest
    when: ansible_distribution == "AlmaLinux"

```

- Create sudoer_simone in files folder: `vi files/sudoer_simone`

```yaml
simone ALL=(ALL) NOPASSWD: ALL
```

- Edit ansible.cfg

```yaml
[defaults]
inventory = inventory
private_key_file = ~/.ssh/ansible
remote_user = simone
```

- bootstrap.yaml
```yaml
---

- hosts: all
  become: yes
  pre_tasks:

  - name: install updates (Ubuntu)
    tags: always
    apt:
      update_cache: yes
      upgrade: dist
    when: ansible_distribution == "Ubuntu"

  - name: install updates (AlmaLinux)
    tags: always
    dnf:
      update_cache: yes
      update_only: yes
    when: ansible_distribution == "AlmaLinux"


- hosts: all
  become: yes
  tasks:

  - name: create simone user
    tags: always
    user:
      name: simone
      groups: root

  - name: add ssh key for simone
    tags: always
    authorized_key:
      user: simone
      key: "ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAILInjsYJAQk+CyEOiPmklZltW4kDeTBuy6DWd5IvWloG ansible"

  - name: add sudoers file for simone
    copy:
      src: sudoer_simone
      dest: /etc/sudoers.d/simone
      owner: root
      group: root
      mode: 0440
```

- site.yaml

```yaml
---

- hosts: all
  become: yes
  pre_tasks:

  - name: update repo (Ubuntu)
    tags: always
    apt:
      update_cache: yes
    changed_when: false
    when: ansible_distribution == "Ubuntu"

  - name: update repo (AlmaLinux)
    tags: always
    dnf:
      update_cache: yes
    changed_when: false
    when: ansible_distribution == "AlmaLinux"


- hosts: workstations
  become: yes
  tasks:

  - name: install unzip
    package:
      name: unzip
      state: latest

  - name: install terraform
    unarchive:
      src: https://releases.hashicorp.com/terraform/0.12.28/terraform_0.12.28_linux_amd64.zip
      dest: /usr/local/bin
      remote_src: yes
      owner: root
      group: root
      mode: 0755


- hosts: web_servers
  become: yes
  tasks:

  - name: install apache and php on Ubuntu
    tags: apache,apache2,web,ubuntu
    apt:
      name:
      - apache2
      - libapache2-mod-php
      state: latest
    when: ansible_distribution == "Ubuntu"

  - name: install apache and php on AlmaLinux
    tags: apache,httpd,web,alma
    dnf:
      name:
      - httpd
      - php
      state: latest
    when: ansible_distribution == "AlmaLinux"

  - name: start and enable httpd service (AlmaLinux)
    tags: apache,httpd,alma
    service:
      name: httpd
      state: started
      enabled: yes
    when: ansible_distribution == "AlmaLinux"

  - name: copy default html file for site
    tags: apache,apache2,httpd
    copy:
      src: default_site.html
      dest: /var/www/html/index.html
      owner: root
      group: root
      mode: 0644


- hosts: db_servers
  become: yes
  tasks:

  - name: install maria db on Ubuntu
    tags: mariadb_packet,mariadb-server,db,ubuntu
    apt:
      name: mariadb-server
      state: latest
    when: ansible_distribution == "Ubuntu"

  - name: install maria db on AlmaLinux
    tags: mariadb_packet,mariadb,db,alma
    dnf:
      name: mariadb
      state: latest
    when: ansible_distribution == "AlmaLinux"


- hosts: file_servers
  become: yes
  tasks:

  - name: install samba on Ubuntu
    tags: samba,file,ubuntu
    apt:
      name: samba
      state: latest
    when: ansible_distribution == "Ubuntu"

  - name: install samba on Alma
    tags: samba,file,alma
    dnf:
      name: samba
      state: latest
    when: ansible_distribution == "AlmaLinux"
```

### 14. Role

- Site with roles
```yaml
---

- hosts: all
  become: yes
  pre_tasks:

  - name: install updates (Ubuntu)
    tags: always
    apt:
      update_cache: yes
    changed_when: false
    when: ansible_distribution == "Ubuntu"

  - name: install updates (AlmaLinux)
    tags: always
    dnf:
      update_cache: yes
    changed_when: false
    when: ansible_distribution == "AlmaLinux"


- hosts: all
  become: true
  roles:
    - base

- hosts: workstations
  become: yes
  roles:
    - workstations

- hosts: web_servers
  become: yes
  roles:
    - web_servers

- hosts: db_servers
  become: yes
  roles:
    - db_servers


- hosts: file_servers
  become: yes
  roles:
    - file_servers
```

- create a roles directory and create a directory for each role you wish to add:

```bash
mkdir roles
cd roles
mkdir base
mkdir db_servers
mkdir file_servers
mkdir web_servers
mkdir workstations
```

- Inside each role directory, create a tasks directory

```
cd <role_name>
mkdir tasks
```

- inside task directory, create a file main.yaml
- Base role main.yml

```yaml
- name: add ssh key for simone
  tags: always
  authorized_key:
    user: simone
    key: "ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAILInjsYJAQk+CyEOiPmklZltW4kDeTBuy6DWd5IvWloG ansible"
```

- main.yaml file for db_servers role

```yaml
- name: install maria db on Ubuntu
  tags: mariadb_packet,mariadb-server,db,ubuntu
  apt:
    name: mariadb-server
    state: latest
  when: ansible_distribution == "Ubuntu"

- name: install maria db on AlmaLinux
  tags: mariadb_packet,mariadb,db,alma
  dnf:
    name: mariadb
    state: latest
  when: ansible_distribution == "AlmaLinux"
```

- main.yaml file for file_servers role

```yaml  
- name: install samba on Ubuntu
  tags: samba,file,ubuntu
  apt:
    name: samba
    state: latest
  when: ansible_distribution == "Ubuntu"

- name: install samba on Alma
  tags: samba,file,alma
  dnf:
    name: samba
    state: latest
  when: ansible_distribution == "AlmaLinux"
```

- main.yaml file for workstations role

```yaml  
- name: install unzip
  package:
    name: unzip
    state: latest

- name: install terraform
  unarchive:
    src: https://releases.hashicorp.com/terraform/0.12.28/terraform_0.12.28_linux_amd64.zip
    dest: /usr/local/bin
    remote_src: yes
    owner: root
    group: root
    mode: 0755
```

- main.yaml file for web_servers role

```yaml  
- name: install apache and php on Ubuntu
  tags: apache,apache2,web,ubuntu
  apt:
    name:
    - apache2
    - libapache2-mod-php
    state: latest
  when: ansible_distribution == "Ubuntu"

- name: install apache and php on AlmaLinux
  tags: apache,httpd,web,alma
  dnf:
    name:
    - httpd
    - php
    state: latest
  when: ansible_distribution == "AlmaLinux"

- name: start and enable httpd service (AlmaLinux)
  tags: apache,httpd,alma
  service:
    name: httpd
    state: started
    enabled: yes
  when: ansible_distribution == "AlmaLinux"

- name: copy default html file for site
  tags: apache,apache2,httpd
  copy:
    src: default_site.html
    dest: /var/www/html/index.html
    owner: root
    group: root
    mode: 0644
```

- tasks copy default html file for site require files folder in roles/web_servers:

```bash
mkdir roles/web_servers/files/
cp files/default_site.html roles/web_servers/files/default_site.html
```

### 15. Host Variables and Handlers

- Create host_vars (ubuntu)

```
apache_package_name: apache2
apache_service: apache2
php_package_name: libapache2-mod-php
```

- host_vars (alma)

```
apache_package_name: httpd
apache_service: httpd
php_package_name: php
```

- Handlers for the web_servers role: create the handlers directory (within the role directory)
roles/web_server/handlers/main.yaml

```yaml
 - name: restart_apache
   tags: apache,centos,httpd
   service:
     name: "Template:Apache service"
     state: restarted
```

- main.yam (web_server role)
  
```yaml
- name: install apache and php package
  tags: apache,httpd,php
  package:
    name:
    - "{{ apache_package_name }}"
    - "{{ php_package_name }}"
    state: latest

- name: start and enable httpd service
  tags: apache,httpd
  service:
    name: "{{ apache_service_name }}"
    state: started
    enabled: yes

- name: change email address for admin on Alma Linux
  tags: apache,alma,httpd
  lineinfile:
    path: /etc/httpd/conf/httpd.conf
    regexp: '^ServerAdmin'
    line: ServerAdmin somebody@somewhere.com
  when: ansible_distribution == "AlmaLinux"
  notify: restart_apache

- name: copy default html file for site
  tags: apache,apache2,httpd
  copy:
    src: default_site.html
    dest: /var/www/html/index.html
    owner: root
    group: root
    mode: 0644
```

##16 Templates