# ansible documentation

```bash
ansible-doc -l       #list modules
ansible-doc ec2      #show aws ec2 module information
```

# Installation

```bash
sudo yum install epel-release
sudo useradd ansible
sud passwd ansible
sudo yum install ansible git
ansible --version  
```

# Create ssh key for server connections
 
```bash
ssh-keygen                                 #generate ssh key on all servers
ssh-copy-id <remote-server, localhost>     #copy ssh key to remote and local server
```

# Allow ansible to use sudo to run any commnad (modify /etc/sudoers)

```bash
sudo visudo
```

Add the following line:
```
ansible ALL=(ALL) NOPASSWD: ALL
```

# Configuration files

``` 
Ansilbe_config        # Environment variable. 1st ansible config file
ansible.cfg           # In current directory
~/.ansible.cfg        # In home directory
/etc/ansible.cfg      # 4th baseline/default config location
ansible-config        # Interact with config file
```

# Configure hosts inventory file

```
/etc/ansible/hosts
```

# Playbook example

```
- hosts: all
  become: yes
  tasks:
    - name: edit host file
      lineinfile:
        path: /etc/hosts
        line: "169.168.0.1 ansible.xyzcorp.com"
    - name: install elinks
      package:
        name: elinks
        state: latest
    - name: create audit user
      user:
        name: xyzcorp_audit
        state: present
    - name: update motd
      copy:
        src: /home/ansible/motd
        dest: /etc/motd
    - name: update issue
      copy:
        src: /home/ansible/issue
        dest: /etc/issue

- hosts: network
  become: yes
  tasks:
    - name: install netcat
      yum:
        name: nmap-ncat
        state: latest
    - name: create network user
      user:
        name: xyzcorp_network
        state: present

- hosts: sysadmin
  become: yes
  tasks:
    - name: copy tarball
      copy:
        src: /home/ansible/scripts.tgz
        dest: /mnt/storage/

- hosts: nfs
  become: yes
  vars:
    share_path: /mnt/nfsroot
  tasks:
    - name: install nfs-utils
      yum:
        name: nfs-utils
        state: latest
    - name: start nfs service
      service:
        name: nfs-server
        state: started
        enabled: yes
    - name: configure exports file
      template:
        src: /home/ansible/exports.j2
        dest: /etc/exports
      notify: update nfs
  handlers: 
    - name: update nfs exports
      command: exportfs -a
      listen: update nfs
 
- hosts: remote
  become: yes
  vars:
    nfs_ip: "{{ hostvars['nfs']['ansible_default_ipv4']['address'] }}" 
    nfs_hostname: "{{ hostvars['nfs']['ansible_hostname'] }}"
  vars_files:
    - /home/ansible/user-list.txt
  tasks:
    - name: configure hosts file
      template:
        src: /home/ansible/etc.hosts.j2
        dest: /etc/hosts.nfslab
    - name: get file status
      stat:
        path: /opt/user-agreement.txt
      register: filestat
    - name: debug info
      debug:
        var: filestat
    - name: create user
      user:
        name: "{{ item }}"
      when:  filestat.stat.exists
      loop: "{{ users }}"

ansible-playbook --ask-vault-pass /home/ansible/webserver.yml
- hosts: webservers
  become: yes
  vars_files:
    - /home/ansible/confidential
  tasks:
    - name: install httpd
      package:
        name: httpd
        state: latest
      notify: httpd service
      tags:
        - base-install
    - name: configure vhosts
      template:
        src: /home/ansible/vhost.conf.j2
        dest: /etc/httpd/conf.d/vhost.conf
      notify: httpd service
      tags:
        - vhosts
    - name: configure site auth
      template:
        src: /home/ansible/htpasswd.j2
        dest: /etc/httpd/conf/htpasswd
      notify: httpd service
      tags:
        - vhost
    - name: run data job
      command: /opt/data-job.sh
      async: 600   #wait 600 seconds for job to complete and cancel if not
      poll: 0       #poll every 0 seconds
      tags:
        - data-job
  handlers:
    - name: restart and enable httpd
      service:
        name: httpd
        state: started
        enabled: yes
      listen: httpd service
```
# Ansible commands

```bash
ansible all --list-hosts                  # Lists all hosts in hosts file
ansible apacheweb -m ping                 # Run ping module against apache web host group
ansible webserverss -i hosts -m ping      # Pass custom hosts file

ansible apacheweb -s -m shell -a 'yum list installed | grep python'
```

# System facts

```bash
ansible local -m setup | more                    # Shows setup facts of local
ansible local -m setup --tree /tmp/facts         # Creates facts file
ansible localhost -m setup -a 'filter=ipv4*'     # Filter ip info
```

# Playbooks

```bash
ansible local -s -m shell -a 'yum install lynx'                             # -s is sudo. this is not a good way of installing
ansible local -s -m yum -a 'pkg=lynx state=installed update_cache=true'     # Install with yum module and check current state
```

# Playbook example (appserver.yml YAML file)

# Task config

```
raw:         #runs raw command
command:     #runs raw command but allows the register and debug commands

register:    #takes output from command:
debug:       #capture register: to variable
async:       #run job in parallel. this is the wait time in milliseconds
poll:        #poll in seconds, used with async

run_once: true     #only runs the command once on the first host

yum: pkg={{ item }} state=present          #this is a playbook task loop
with_items:
 - lynx
 - telnet

when: ansible_os_family == "debian"     #runs a task based on condition

shell: systemctl status httpd          #check for active httpd daemon using until loop
register: result
until: result.stdout.find("active (runnning)") != -1
retries: 5
delay: 5          #delay in seconds
debug: var=result     #display result

action: yum name=https state=installed          #use notify to execute a command when a change is detected. in this example, httpd is restarted only if Apache is not installed
notify: restart httpd     #must match handler name
handlers:
 - name: restart httpd
    action: service name=httpd state=restarted

tags:          #Runs specific playbook section. ansible-playbook tag.yml --tags "packages" or --skip-tags "packages"
 -packages

tags:
 - always     #always run section unless skipped

local_action:          #redirect command to control server
delegate_to:          #redirect command to specified server
```

# Ansible vault

```bash
ansible-vault create secure.yml      #creates and encrypted yaml file for storage secrets
ansible-vault edit secure.yml
ansilbe-vault rekey secure.yml       #change vault password
ansible-vault decrypt secure.yml     #descrypt yaml file
ansible-vault encrypt secure.yml     #encrypt yaml file
ansible-playbook --ask-vault-ass     #prompt for vault file password if file is linked in playbook

ansible-playbook --vault-file-password file.yml     #pass in file
```

# Includes; Used with 'plays'

```
play.yml     #minimal markup required
- name: install telnet
  yum: pkg-telnet state=installed

playbook.yml
tasks:
 - include: plays/play.yml
```

# Start at task

```bash
ansible-playbook playbook.yml --start-at-task=='task name'     #start at specific task
ansible-playbook playbook.yml --step                           #step through all tasks
```

# Pass in variable from command line

```bash
ansible-playbook playbook.yml --extra-vars "hosts=webservers pkg=lynx"
```

# Roles

```
ansible-galaxy init /home/ansible/role-name
```
```
roles directories: defaults, files, handlers, meta, tasks, templates, vars

main.yml          #this is created in the roles directories

webesrvers.yml      #master playbook that runs through the webservers role directory and runs each play/main.yml
```

```
tasks:
  - name: use web role
     include_role:
       name: /home/ansible//web

pre_tasks          #used in playbook to run tasks before role
post_task          #used in playbook to run tasks after role
```