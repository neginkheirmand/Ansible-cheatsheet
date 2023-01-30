# ansible cheat-sheet:







[TOC]



## things you should probably do when installing ansible:

1- create a ssh key **just for ansible** to use, you can check [this](https://www.youtube.com/watch?v=-Q4T9wLsvOQ&list=PLT98CRl2KxKEUHie1m24-wkyHpEsa4Y70&index=3) video for more info.

ssh-keygen -t ed25519 -C 'ansible default'

![image-20230124050634187](https://github.com/neginkheirmand/Ansible-GettingStarted/blob/master/getting-started/doc/1.png?raw=true)

Remember to not put any passphrase for it. I would also recommend putting it into a different file (so that its only for ansible to use) as shown in the image above.

now you should copy the ssh key created to the said server:

`ssh-copy-id -i ~/.ssh/ansible.pub 10.24.34.57`

![image-20230124051643716](https://github.com/neginkheirmand/Ansible-GettingStarted/blob/master/getting-started/doc/2.png?raw=true)



## how to install ansible:

 ### control node requirements:

Basically be it any OS  linux-based, or just WSL for windows, and you need some newer version of python installed.

for more info visit [this](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html#id6) ansible installation documentation page.

### Managed node requirements

The managed node needs any newer version of python installed and also a user account that can SSH to the node with an interactive POSIX shell.



### Installation:

#### Using pip

check that you have pip installed

```bash
python3 -m pip -V
curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py
python3 get-pip.py --user
python3 -m pip install --user ansible
```
check whether it was correctly installed:

```bash
ansible --version
```

#### Using apt

```bash
sudo apt-add-repository ppa:ansible/ansible
```





## Command(AD HOC) cheat-sheet:

ping all machines:
```bash
#remember to always use command ssha before using ansible, since that will set the ssh-agent in work 

#to ping all the servers 
ansible all -m ping 

#to ping all the server from the "virtualmachines" group according to the inventory config 
ansible virtualmachines -m ping -i inventory.yaml

#Show Ansible inventory information, by default it uses the inventory script JSON format
ansible-inventory -i inventory.yaml --list


ansible all -a "df -h" -u root
ansible-inventory --list -y
ansible all --list-hosts

# run the gather_facts command on the destination server 
ansible all -m gather_facts --limit <destination>

#check playbook syntax
ansible-playbook  create_users.yml  --syntax-check

```



## AD HOC vs playbook: 

![51](https://github.com/neginkheirmand/Ansible-GettingStarted/blob/master/getting-started/doc/51.png?raw=true)



## Playbooks

### Variables

use "when" and the variables from **gather_facts** command to conditionally specify which server you want your plays to run at:

#### gather_facts variables

`ansible all -m gather_facts --limit <destination>`

it should look something like this:

![image-20230128144305205](https://github.com/neginkheirmand/Ansible-GettingStarted/blob/master/getting-started/doc/3.png?raw=true)

 for example for the ansible_distribution variable:

![image-20230128144406326](https://github.com/neginkheirmand/Ansible-GettingStarted/blob/master/getting-started/doc/4.png?raw=true)

and use it in a play like this:

![image-20230128144451878](https://github.com/neginkheirmand/Ansible-GettingStarted/blob/master/getting-started/doc/5.png?raw=true)

 #### self-defined variables:

![image-20230128151606816](https://github.com/neginkheirmand/Ansible-GettingStarted/blob/master/getting-started/doc/6.png?raw=true)

![image-20230128151752587](https://github.com/neginkheirmand/Ansible-GettingStarted/blob/master/getting-started/doc/7.png?raw=true)



### Tags:

#### general and self-defined

general: like always (will run no matter what tag we targeted)

self-defined: like ubuntu in this next example

![image-20230128172305176](https://github.com/neginkheirmand/Ansible-GettingStarted/blob/master/getting-started/doc/8.png?raw=true)

and to list all the tags from a playbook:

` ansible-playbook --list-tags install_latest_tree_package.yaml`

![image-20230128172525586](https://github.com/neginkheirmand/Ansible-GettingStarted/blob/master/getting-started/doc/9.png?raw=true)

to run while targeting a specific tag :

`ansible-playbook --tags ubuntu -K install_latest_tree_package.yaml`

to run while targeting many tag :

`ansible-playbook --tags "ubuntu,db" -K install_latest_tree_package.yaml`



### copy files to server and changing UFW config:



![image-20230129132050766](https://github.com/neginkheirmand/Ansible-GettingStarted/blob/master/getting-started/doc/10.png?raw=true)



![image-20230129131023769](https://github.com/neginkheirmand/Ansible-GettingStarted/blob/master/getting-started/doc/11.png?raw=true)

![image-20230129131049641](https://github.com/neginkheirmand/Ansible-GettingStarted/blob/master/getting-started/doc/12.png?raw=true)



more on changing UFW rules and status [**here**](https://docs.ansible.com/ansible/latest/collections/community/general/ufw_module.html) and its `examples`

### installing applications using unzip:

![image-20230129133821356](https://github.com/neginkheirmand/Ansible-GettingStarted/blob/master/getting-started/doc/13.png?raw=true)



### managing services:

above we have an example of installation of apache, as you will later see, this service will automatically start after the installation but in many Operating systems -for example CentOS- (and services other than apache) this may not occur, so we have to have an specific play for starting services, we may later also need to know how to stop, ... services.

#### starting service:

version1:

![image-20230129135955709](https://github.com/neginkheirmand/Ansible-GettingStarted/blob/master/getting-started/doc/14.png?raw=true)

just to install and start the httpd (apache for centos) after running the playbook

- in this setup after logging to the centos server we can see that httpd is running however since its **disabled**, after a reboot, we will again need to run this playbook to again start the service. to fix this, the version 2 of this playbook did the next:

#### enabling service(start running after reboot)

![image-20230129135955709](https://github.com/neginkheirmand/Ansible-GettingStarted/blob/master/getting-started/doc/15.png?raw=true)

and this is a very simple thing to fix, we just need to add one line:

![image-20230129144359411](https://github.com/neginkheirmand/Ansible-GettingStarted/blob/master/getting-started/doc/16.png?raw=true)

 and after running the playbook with this new line, the service should look likes this:

![image-20230129144359411](https://github.com/neginkheirmand/Ansible-GettingStarted/blob/master/getting-started/doc/17.png?raw=true)

#### restarting a service

for example we change this line from the "/etc/httpd/conf/httpd.conf" file and after it we should restart the httpd (apache for centos) package, so the change is applied:

![image-20230129144359411](https://github.com/neginkheirmand/Ansible-GettingStarted/blob/master/getting-started/doc/18.png?raw=true)

we need to know exactly what line we will be changing:

![image-20230129144359411](https://github.com/neginkheirmand/Ansible-GettingStarted/blob/master/getting-started/doc/19.png?raw=true)

#####  **WARNING: lineinfile package**

**Be very carefull when using the lineinfile module because this change may happen everytime that the playbook runs, because the playbooks doesn't know that you have already ran this**, for example:

 you may want to add "*Aa12345*" after the word "*hi*" the line 5, what happens is Aa12345 will be added the amount of times you run this playbook,

 cause the playbook cant know that has already be ran before and you will end up with something like this: 

*hi Aa12345Aa12345Aa12345Aa12345*

 **but this will not happen for most of the other plays, for example installation ... , since something has already been installed, the play wont change a thing.**



![image-20230129144359411](https://github.com/neginkheirmand/Ansible-GettingStarted/blob/master/getting-started/doc/20.png?raw=true)

before and after running this playbook:



![image-20230129155508819](https://github.com/neginkheirmand/Ansible-GettingStarted/blob/master/getting-started/doc/21.png?raw=true)



### managing users:

#### creation: 

see a list of users with this command: 	`cat /etc/passwd`

as of right now the latest added users are:

![image-20230129170932955](https://github.com/neginkheirmand/Ansible-GettingStarted/blob/master/getting-started/doc/2.png?raw=true)

run the playbook to add users :

![image-20230129171146097](https://github.com/neginkheirmand/Ansible-GettingStarted/blob/master/getting-started/doc/24.png?raw=true)

and this should be the output of `cat /etc/passwd` after running the playbook:

![image-20230129171115472](https://github.com/neginkheirmand/Ansible-GettingStarted/blob/master/getting-started/doc/23.png?raw=true)

#### add sshkey for said user:

![image-20230129171428118](https://github.com/neginkheirmand/Ansible-GettingStarted/blob/master/getting-started/doc/25.png?raw=true)

![image-20230129171810575](https://github.com/neginkheirmand/Ansible-GettingStarted/blob/master/getting-started/doc/26.png?raw=true)

#### add sudoers file for said user

![image-20230129172654747](https://github.com/neginkheirmand/Ansible-GettingStarted/blob/master/getting-started/doc/27.png?raw=true)

to test it:

![image-20230129172833553](https://github.com/neginkheirmand/Ansible-GettingStarted/blob/master/getting-started/doc/28.png?raw=true)

and should be known as simone and be able to use sudo:

![image-20230129172950638](https://github.com/neginkheirmand/Ansible-GettingStarted/blob/master/getting-started/doc/29.png?raw=true) 



### Roles

Roles allow us to better split up our tasks.

you should define the roles in the playbook.yml and then create a **"roles"** folder in the same path as the playbook, inside of the roles folder there should be folder for each of the roles defined in the playbook, named with their respective role_names.

![image-20230130125029635](https://github.com/neginkheirmand/Ansible-GettingStarted/blob/master/getting-started/doc/30.png?raw=true) 

each role_folder should have a tasks folder inside:

![image-20230130125337800](https://github.com/neginkheirmand/Ansible-GettingStarted/blob/master/getting-started/doc/31.png?raw=true)



and inside the tasks directory corresponding to each role there should be a main.yml that is called a taskbook instead of a playbook, because it should start with tasks instead of host specifications and  ...

![image-20230130133741890](https://github.com/neginkheirmand/Ansible-GettingStarted/blob/master/getting-started/doc/32.png?raw=true)

* remember if you are coping files and you need the **"files"** directory you should also copy the files directory into the said role_folder

![image-20230130134302367](https://github.com/neginkheirmand/Ansible-GettingStarted/blob/master/getting-started/doc/33.png?raw=true)



### Host variables and handlers

to create varibles for each of our hosts, we can create a hosts_vars directory and  create files named after the host(ip, dns or domain name of the hosts -> basically whatever you are calling it in the inventory file with .yml file-extension)

we did this with the webservers:

![image-20230130140908417](https://github.com/neginkheirmand/Ansible-GettingStarted/blob/master/getting-started/doc/34.png?raw=true)

![image-20230130140952764](https://github.com/neginkheirmand/Ansible-GettingStarted/blob/master/getting-started/doc/35.png?raw=true)





### Handlers

do you remember when we did this:

![image-20230130140952764](https://github.com/neginkheirmand/Ansible-GettingStarted/blob/master/getting-started/doc/36.png?raw=true)

we used variable named **apache** to register when a changed happened in the state, but as mentioned before this is not very clever, because you can possibly only use that variable in one time: 

* you could handle this scenario:
  * when task A is done -> do task B 
    * ​	A -> B
* but you couldn't easy handle this scenario:
  * when task A is done -> do task B and when B is done -> do task C
    * **to do this you would have to use 2 variables:**
      * A -> B
      * B -> C 

but there is an easier way to handle this scenario without using variables:

we will completely remove the restart task and the variables like this:

![image-20230130141549553](https://github.com/neginkheirmand/Ansible-GettingStarted/blob/master/getting-started/doc/52.png?raw=true)



we will now create a folder in the said directory where this taskbook existed and call it handlers, with a main.yml file inside of it and inside of it the task that should run with the name of the notification should exists:

![image-20230130142137922](https://github.com/neginkheirmand/Ansible-GettingStarted/blob/master/getting-started/doc/38.png?raw=true) 

and the main taskbook should look like this:

![image-20230130142137922](https://github.com/neginkheirmand/Ansible-GettingStarted/blob/master/getting-started/doc/39.png?raw=true)



### Templates

 the ssh agent has some configuration. the path to this config file(***/etc/ssh/sshd_config***) is the same across different distributions of linux however the contents may vary. Sometimes you want this config to be the same across the different servers that are under your management, for this, you can use templates:

we will  create a copy of this file and save it with the same name (*sshd_config*) but with a j2 file extension(jinja2)

![image-20230130150209748](https://github.com/neginkheirmand/Ansible-GettingStarted/blob/master/getting-started/doc/40.png?raw=true)

![image-20230130150334180](https://github.com/neginkheirmand/Ansible-GettingStarted/blob/master/getting-started/doc/41.png?raw=true)



first we check that AllowUsers doesn't already exist in the file:

![image-20230130150529736](https://github.com/neginkheirmand/Ansible-GettingStarted/blob/master/getting-started/doc/42.png?raw=true)

and then we add it to the sshd_config_ubuntu.j2 and sshd_config_centos.j2 file:

![image-20230130150529736](https://github.com/neginkheirmand/Ansible-GettingStarted/blob/master/getting-started/doc/43.png?raw=true)

and now will set the value of this variable:

![image-20230130151011973](https://github.com/neginkheirmand/Ansible-GettingStarted/blob/master/getting-started/doc/44.png?raw=true)

![image-20230130151118227](https://github.com/neginkheirmand/Ansible-GettingStarted/blob/master/getting-started/doc/45.png?raw=true)

we will now add it to our host vars for each of the servers

![image-20230130151118227](https://github.com/neginkheirmand/Ansible-GettingStarted/blob/master/getting-started/doc/46.png?raw=true)

and now add a base task(for every server) to add this file to their respective ssh config path file:

![image-20230130152314037](https://github.com/neginkheirmand/Ansible-GettingStarted/blob/master/getting-started/doc/47.png?raw=true)

and since this notified a handler we should create one for it:

![image-20230130152353616](https://github.com/neginkheirmand/Ansible-GettingStarted/blob/master/getting-started/doc/48.png?raw=true)



![image-20230130152439328](https://github.com/neginkheirmand/Ansible-GettingStarted/blob/master/getting-started/doc/49.png?raw=true)

![image-20230130152528658](https://github.com/neginkheirmand/Ansible-GettingStarted/blob/master/getting-started/doc/50.png?raw=true)

after running this, the /etc/ssh/sshd_config file for each server should have an extra line (which is the AllowUsers one)



---



# playbook examples in order of complexity 

1. **install_latest_tree_package.yaml**
2. **install_zip.yml**
3. **uninstall_tree_package.yaml**
4. **print_arr.yml**
5. **install_php_and_apache_on_ubuntu_and_centos.yml**
6. **copy-files-to-server.yml**
7. **install-terraform.yml**
8. **change_httpd_config_and_restart.yml**
9. **create_users.yml**
10. **start_httpd_centos.yml**
11. roles playbook is inside the **role-example** folder

