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

