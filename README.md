# Ansible-for-DevOps
Ansible for DevOps

Imp CMD:
--------

**To ping a specific group**: 

     ansible group_name -i inventory -m ping

**Syntax for creating a parent_group** 
 
    [parent_group_name:children]

**To ping a parent_group**: 

    ansible parent_group_name -i inventory -m ping

**How do you visualize an inventory**: 

    ansible-inventory -i inventory --graph

**host-variables & group-variables**:

    host-variables are spec of single host. group-variable are same specifications which are useing by the multiple hosts.


Manual SSH:

    ssh -i /root/ansible-lab/key.pem ec2-user@172.31.79.2

Check SSH service: 

     sudo systemctl status sshd

Check port 22:

     nc -vz 172.31.79.2 22

Ansible ping:

    ansible server01 -i inventory -m ping

Execute uptime: 

    ansible server01 -i inventory -m command -a "uptime"

Execute hostname:

    ansible server01 -i inventory -m command -a "hostname"

Create a folder in the host/group:

        ansible server01 -i inventory -m file \
       -a "path=/tmp/ansible-test state=directory"

Check the file is created or not in the host-path:

     ansible server01 -i inventory -m command \
       -a "ls -ld /tmp/ansible-test"

create the file:

        ansible server01 -i inventory -m file \
       -a "path=/tmp/ansible-test/test.txt state=touch"

Copy a File:

     ansible server01 -i inventory -m copy \
       -a "src=/root/ansible-lab/test.txt dest=/tmp/ansible-test/test.txt"

To wrire a file:

     ansible server01 -i inventory -m shell -a "echo "hello" >> /root/file.txt" -b

To Read a File:

     ansible server01 -i inventory -m command -a "cat /root/file.txt" -b

Install Nginx:

     ansible server01 -i inventory -m dnf \
       -a "name=nginx state=present" -b

To Check the software version:

     ansible server01 -i inventory -m command \
       -a "nginx -v"

To Start Nginx:

     ansible server01 -i inventory -m systemd \
       -a "name=nginx state=started" -b

       with command

       ansible server01 -i inventory -m command \
       -a "systemctl is-active nginx"

To Enable Nginx:

     ansible server01 -i inventory -m systemd \
       -a "name=nginx state=started enabled=yes" -b

Test Idempotency: 

Run the installation command again:

     ansible server01 -i inventory -m dnf \
       -a "name=nginx state=present" -b

Because Ansible checks:

     Is Nginx installed?
            |
           YES
            |
            v
     No change required

Check Everything:

      ansible server01 -i inventory -m command \
       -a "nginx -v"

       ansible server01 -i inventory -m command \
       -a "systemctl is-active nginx"

       ansible server01 -i inventory -m command \
       -a "systemctl is-enabled nginx"

