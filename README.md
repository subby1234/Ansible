# Ansible
USING UBUNTU
python3 is already in-built

For other O/S 

sudo apt-get install python3 ........ FOR BOTH MASTER AND SLAVE




2. Update machine:
sudo apt update

3. Go to the documentation of ansible:
https://docs.ansible.com/ansible/latest/installation_guide/installation_distros.html#installing-ansible-on-ubuntu
Run the command on master
sudo apt install software-properties-common
sudo add-apt-repository --yes --update ppa:ansible/ansible
sudo apt install ansible

4. Generate a SSH KEY ON MASTER
Ssh-keygen
Hit enter 3 times
sudo cat /home/ubuntu/.ssh/id_rsa.pub
Copy SSH key from master

5. Go to test machine
Cd .ssh>ls>sudo nano authorized_keys
Paste the ssh key >Ctrl +s >Ctrl+X


7. Go to master machineCd /etc/ansible
Ls
Sudo nano hosts
[slave]
172.31.9.93
172.31.9.54


(Paste the private ip of both slaves)
Slave1 ansible_host=172.31.9.93
Slave2 ansible_host=172.31.0.169
Ctrl+s and ctrl+x

8. ansible -m ping all




SSH-KEYGEN ----------on master
copy the public key from master to slave
MASTER-------cat ./.ssh/id_rsa.pub
copy it....

IN SLAVE........
sudo nano ./.ssh/authorized_keys
past it in here...in line 2

ssh@ubuntu@slave IP




MASTER
sudo apt update
sudo apt install software-properties-common
sudo apt-add-repository --yes --update ppa:ansible/ansible
sudo apt install ansible


We need to put/save IP in the hosts of the master using:
sudo nano /etc/ansible/hosts

[servers]
slave1 ansible_host=IP
slave2 ansible_host=IP

ansible -m ping all


CREATING PLAYBOOK
mkdir ansible
cd ansible
sudo nano play.yml


---
- hosts: slave1
  become: yes
  name: Installing nginx on slave1
  tasks:
   - name: Installing
     apt: name=nginx state=latest

- hosts: slave2
  become: yes
  name: Installing apache on slave2
  tasks:
   - name: Installing
     apt: name=apache2 state=latest

cd /etc/ansible/roles
ansible-playbook service.yml




EXECUTING SCRIPT ON PLAYBOOK

sudo nano script.sh

                                           script.sh *                                                            
#!/bin/sh
echo "hello world" > /var/www/html/1.html


adjust : sudo nano play.yml
---
- hosts: slave1
  become: yes
  name: Installing nginx on slave1
  tasks:
   - name: Installing
     apt: name=nginx state=latest
   - name: Running a script
     script: script.sh

- hosts: slave2
  become: yes
  name: Installing apache on slave2
  tasks:
   - name: Installing
     apt: name=apache2 state=latest

ansible-playbook play.yml









ANSIBLE ROLE
/etc/ansible/roles

ansible-galaxy init role-name 

..install tree
sudo apt install tree

...to view the tree
tree role-name

STEP 1: GO INSIDE TASK and edit
/etc/ansible/roles/apache/tasks

sudo nano main.yml

DIVIDE the tasks into 3:

---
- include: install.yml
- include: configure.yml
- include: service.yml

Update the 3 tasks



sudo nano install.yml

---
- name: install apache2
  apt: name=apache2 update_cache=yes state=latest
  become: true
  


sudo nano configure.yml

---
- name: apache2.conf file
  copy: src=apache2.conf dest=/etc/apache2
  become: true
  notify: -restart apache2 service
- name: send copy.html file
  copy: src=copy.html dest=/home/ubuntu/
  become: true



sudo nano service.yml

---
- name: starting apache2 service
  service: name=apache2 state=latest
  become: true
  
  
  
STEP 2: STORE THE FILE THAT NEEDED TO BE PUSHED TO REMOTE MACHINE
ubuntu@ip:$ cp /etc/apache2/apache.conf /etc/ansible/roles/apache/files
cp /etc/apache2/apache.conf /etc/ansible/roles/apache/files


CREATE HTML FILE AS WELL
/etc/ansible/role/apache/files

sudo nano copy.html

<html>
  <head>
    <title>some file</title>
  </head>
  <body>
    <h1>copy this file</h1>
  </body>
</html>


ls
apache2.conf copy.html



STEP 3: GO INSIDE HANDLER and edit
CD /etc/ansible/roles/apache/handles/

sudo nano main.yml

---
- name: install apache2
  apt: name=restart service
  service: name=apache2 state=started
  
  
 
STEP 4: GO INSIDE META and edit
CD /etc/ansible/roles/apache/meta/
  

sudo nano main.yml
galaxy-info:
   author: Intellipaat
   description: simple apache role
   company: Intellipaat
   
   
   
FINALLY. Create top level yml file to add hosts and roles
cd /etc/ansible

sudo nano site.yml

---
- hosts: hosts1
  roles:
    - apache

Execute the top level .yml file

ansible.playbook site.yml --syntax-check





USING ANSIBLE ROLE IN PLAYBOOK

/etc/ansible

sudo nano playbookrole.yml

---
- hosts: host1
  become: true
  tasks:
    - debug:
        msg: "before we run our role"
    - import_role:
        name: apache
    - debug:
        msg: "after we ran our role"





Another example of an Ansible playbook that includes a role. This example assumes you have a custom role named nginx:

yaml

---
- hosts: host1
  become: true
  tasks:
    - debug:
        msg: "Before we run our role"

    - name: Include the nginx role
      import_role:
        name: nginx

    - debug:
        msg: "After we ran our role"

In this example:

    The playbook is applied to hosts in the web_servers group.
    It includes three tasks:
        A debug message before running the role.
        The nginx role is imported and executed on the target hosts.
        Another debug message after running the role.

Make sure to replace web_servers/host1 with the actual group of hosts where you want to apply this playbook, and ensure that the nginx role is defined and available in your Ansible roles directory.



............................................................................
1. Create 2 Ansible roles
2. Install Apache2 on slave1 using one role and NGINX on slave2 using the
other role
3. Above should be implemented using different Ansible roles

MAKE 2 ROLES:APACHE AND NGINX

cd /etc/ansible/roles
sudo ansible-galaxy init apache

cd apache/tasks
sudo nano main.yml

---
- name: Install Apache2
  apt:
    name: apache2
    state: present

.................................................

cd /etc/ansible/roles
sudo ansible-galaxy init nginx

cd nginx/tasks.......................cd /etc/ansible/roles/nginx/tasks
sudo nano main.yml

---
- name: Install NGINX
  apt:
    name: nginx
    state: present
.........................................

Create the service.yml Playbook:

cd /etc/ansible/roles
sudo nano service.yml

---
- name: Install Services
  hosts: all
  become: true
  roles:
    - role: apache
      when: inventory_hostname == 'slave1'
    - role: nginx
      when: inventory_hostname == 'slave2'


ansible-playbook service.yml
.......................................................................

1. Use the previous deployment of Ansible cluster
2. Configure the files folder in the role with index.html which should be
replaced with the original index.html
All of the above should only happen on the slave which has NGINX installed
using the role.


.......................................

In the directory NGINX Ansible role; create a files directory and ..to place the index.html file in it.
cd /etc/ansible/roles/nginx
mkdir files "already exists"
sudo nano files/index.html
............................

Add your custom index.html content and save the file.

<!DOCTYPE html>
<html lang="en">
<head>
     <title>Welcome to My Website</title>
</head>
<body>
    <h1>Hello, Ansible!</h1>
    </html>   
  .......................................    
  
  Modify the main.yml file in the tasks directory of the nginx role to copy the index.html file to the appropriate location.


sudo nano /etc/ansible/roles/nginx/tasks/main.yml
cd /etc/ansible/roles/nginx/tasks : sudo nano main.yml

update it with:
---
- name: Copy custom index.html on slave with NGINX
  become: yes
  copy:
    src: files/index.html
    dest: /usr/share/nginx/html/index.html
  when: "'slave2' in inventory_hostname"
..........................................

to become:
---
- name: Install NGINX
  apt:
    name: nginx
    state: present
- name: Copy custom index.html on slave with NGINX
  become: yes
  copy:
    src: files/index.html
    dest: /usr/share/nginx/html/index.html
  when: "'slave2' in inventory_hostname"

........................................................................


Run your playbook, which includes both the apache_role and nginx_role, on both hosts.

cd /etc/ansible/roles

ansible-playbook service.yml
...........................................................


TO CHECK:
ssh username@slave2_ip
cd /usr/share/nginx/html/
cat index.html


....................................................................................................
1.create a new deployment of Ansible cluster of 5 nodes
2. Label 2 nodes as test and other 2 as prod
3. Install Java on test nodes
4. Install MySQL server on prod nodes
Use Ansible roles for the above and group the hosts under test and prod.

sudo nano /etc/ansible/hosts

[test]
test-node1 ansible_host=54.161.135.205
test-node2 ansible_host=54.211.200.169

[prod]
prod-node3 ansible_host=54.166.156.212
prod-node4 ansible_host=3.92.45.111

ansible -m ping all
......................................

CREATE THE 2 ROLES

/etc/ansible/roles

sudo ansible-galaxy init java_install
sudo ansible-galaxy init mysql_install
...................................


mkdir -p roles/java_install/tasks


.........................................
mkdir -p roles/java_install/tasks..... is used to create the directory structure for an Ansible role. 
sudo nano roles/java_install/tasks/main.yml..... to edit the main.yml file inside the tasks directory.

OR
cd /etc/ansible/roles/java_install/tasks

# Edit the main.yml file
sudo nano main.yml

.........................................

---
- name: Install Java
  become: yes
  apt:
    name: openjdk-11-jdk  
    state: present  
................................


mkdir -p roles/mysql_install/tasks ....... 
sudo nano roles/mysql_install/tasks/main.yml

OR
cd /etc/ansible/roles/mysql_install/tasks
sudo nano main.yml
---
- name: Install MySQL Server
  become: yes
  apt:
    name: mysql-server
    state: present

...................................

Create the service.yml Playbook:

cd /etc/ansible/roles
sudo nano service.yml

---
- name: Install Services
  hosts: all
  become: yes
  roles:
    - role: java_install
    - role: mysql_install


cd /etc/ansible/roles
ansible-playbook service.yml
