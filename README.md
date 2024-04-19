## Deploying a Web Server with Ansible

*Introduction:* In this article, we'll explore how to automate the deployment of a web server using Ansible. Ansible is a powerful open-source automation tool that simplifies the process of managing and deploying infrastructure.

*Prerequisites:* Before we begin, ensure you have the following 
1. Azure account and subscription 

2. Basic familiarity with Linux and Ansible.

3. GitHub where the source code for the webapp resides 

## Things to be covered in Hands-On.

a. Create a host server with any name of your choice (ansible-hostVm)

b. Install Ansible to Host server.

c. Create one more instance server with any name of your choice (answebserverVm)

e. Locking down your files

f. Setup an inventory file and add answebserverVm in the inventory

g. Configure the servers using command method.

h. Uses of playbooks.

i. Deployment of a simple webpage using Ansible.

## Steps to follow:
1. create two Ubuntu server VMs with same key pair, one server will server as the host server and the other as webserver or target server 
 
2. SSH into the Host server and give sudo privilege to it

'visudo' command is a special tool in Unix/Linux for safely updating, providing and managing privileged access in the /etc/sudoers file. 

![image](https://github.com/akpatiudo/mico-ansibleproject/assets/118566096/9fad2ec2-b5d0-4beb-8297-e5908aa75780)

3. Download package information from all configured sources and install ansible.

sudo apt update

sudo apt install ansible

ansible - -version

4. Create your public and private keys which will be used to access the webserver from the Ansible host. on the terminal run the command and follow the prompt to generate the ssh keys.

ssh-keygen # will generate the keys

cd .ssh/  # let you into home directory (/home/username/.ssh)

ls       # will list the files in .shh directory

cat id_rsa.pub # will expose the ssh key generated

5. Copy the content of the public key from the host server into .ssh/authorized_keys file on the web server, past it on a new line

6. Use the command ssh and try to ping the webserver.

![image](https://github.com/akpatiudo/mico-ansibleproject/assets/118566096/1dd2df89-8cc6-436b-bd50-e1b01b289bae)

7. Locking Down Your Files: Understanding chmod Permissions

The chmod command controls file access on Linux systems, grant access level privileges to users. Numbers like 400, 600, and 700 represent permission levels. Here's a breakdown:

The First Digit: Defines owner permissions: Read-only access

The Second Digit: Defines group permissions (often set to 0 for non-shared use)

The Third Digit: Defines other (everyone else) permissions (often set to 0 for maximum security)
Common Levels Explain

400 (r - - - - - ): Only the owner can read the file. Ideal for highly sensitive data.

600 (rw - - - - ): Owner has read and write access, but no one else can access the file. Good for private configuration files.

700 (rwx - - - ): Owner has full control (read, write, execute), but no access for others. Useful for scripts or executables the owner needs exclusive control over.

My generated key pair was upload into my /home/ebenezer in azure, this is to enable me copy it seamlessly when needed. I gave access level of chmod 400, only the owner can read.

Remember: Choose the chmod level that balances security with the intended use of the file.

8. Copy the private key from azure /home/ebenezer into  Host server (ansible-hostVm) at (/home/ansadminuser/). you can use the following command to copy the key.

scp -i "<<key pair name>>" <<key pair name>> <<username@ip>>:/home/username/.ssh

![image](https://github.com/akpatiudo/mico-ansibleproject/assets/118566096/d0a974f2-a85d-4d87-a347-d3327511c0fa)

9. Create an inventory file, in this work I will use the default

sudo vim /etc/ansible/hosts  # it will open the inventory file on a text editor
This was added to the file and saved:

[webserver]

answebserverVm ansible_host=4.223.175.185 ansible_user=webuser1

[ansible_host]

4.223.175.150 ansible_user=ansadminuser ansible_ssh_private_key_file=/home/ansadminuser/ansiblemico-key.pem

ansible -i inventory webserver -m ping   # run this command

![image](https://github.com/akpatiudo/mico-ansibleproject/assets/118566096/760ef0e9-4b62-445f-be78-3fe2575847e9)

This output demonstrates that your Ansible inventory file is configured correctly, and you can successfully target and manage both your Ansible control machine and the web server node defined in the webservers group using Ansible modules.

With successful communication established, you can now explore using Ansible playbooks to automate tasks on your web server node. 

10. Create a playbook. A playbook is a file used to run tasks with Ansible.

The playbook below shows the differnt tasks that will be executed.

The tasks include:

create a group

create a user

create a file

create a webfolder

install apache2

start apache service

install unzip

decompress a file

copy the file into /var/www/html

Note: the play book will download css template from GitHub with the url specified, the template was from www.free-css.com.

nano my_playbook.yml # will create the file My_playbook.yml and open a text editor 

---
- hosts: webserver
  become: true
  tasks:
    - name: Create Group
      group:
        name: webadmin
        gid: 1001

    - name: Create User
      user:
        name: webuser
        state: present
        uid: 2000   # Choose a unique UID here
        group: webadmin
        shell: /bin/bash

    - name: Create web root directory
      file:
        path: /var/www/html
        state: directory
        owner: webuser
        group: webadmin
        mode: '0755'

    - name: Install Apache2
      apt:
        name: apache2
        state: present

    - name: Start the apache2 service
      service:
        name: apache2
        state: started
        enabled: yes

    - name: Install unzip
      apt:
        name: unzip
        state: present

    - name: Decompress the downloaded CSS template
      unarchive:
        src: https://github.com/akpatiudo/mico-ansibleproject/archive/main.zip
        dest: /var/www/html
        remote_src: yes
        owner: webuser
        group: webadmin
        mode: '0755'

    - name: Copy downloaded content into /var/www/html
      shell: |
        cd /var/www/html && cp -r mico-ansibleproject-main/* /var/www/html/

11. Run the ansible playbook

ansible-playbook -i inventory my_playbook.yml

![image](https://github.com/akpatiudo/mico-ansibleproject/assets/118566096/ba3469da-9eb9-41a0-bb1e-35cca23e007b)

12. Testing: Open your browser and type in the public ip of your server. You should see a page similar to the ones below. make sure that port 80 is open in your webserver
Conclusion: In this article, we've demonstrated how to deploy a web server using Ansible. By automating infrastructure provisioning and configuration, Ansible streamlines the deployment process and ensures consistency across environments. With Ansible, managing complex IT infrastructure becomes simpler and more efficient.

![image](https://github.com/akpatiudo/mico-ansibleproject/assets/118566096/b047d490-fd53-49bd-a78d-03cd921a3c34)

Additional Resources:

Ansible Documentation: https://docs.ansible.com/ansible/latest/index.html

Ansible Galaxy (Community Playbooks): https://galaxy.ansible.com/
