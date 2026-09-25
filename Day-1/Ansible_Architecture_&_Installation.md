Ansible Architecture & Installation
=====================================


What is Ansible?
----------------
Ansible is an automation and configuration-management tool used to automate tasks on multiple servers.

    Install packages
    Deploy an application
    Copy configuration files
    Restart services
    Patch servers
    Reboot servers
    Check application health


Ansible Architecture
---------------------

The most important concept today is:

                 CONTROL NODE
                 Ansible
                    |
          +---------+---------+
          |                   |
          v                   v
     Linux Servers       Windows Servers
          |                   |
         SSH                 WinRM


Control Node: The machine where Ansible is installed.

Managed Nodes: The servers that Ansible manages.


Agentless Architecture
-----------------------

* One major advantage of Ansible is that you normally don't install an Ansible agent on every managed server. 
* Ansible generally avoids that additional agent-management requirement.

How Ansible Communicates with Linux
------------------------------------

The basic flow is:


                    Ansible Controller
                           |
                           |
                         SSH
                           |
                           v
                    Linux Server
                           |
                    Execute module
                           |
                           v
                       Result
                           |
                           v
                    Ansible Controller


For example, Ansible might connect as:

     ansible → SSH → ubuntu@10.0.2.10


How Ansible Communicates with Windows
--------------------------------------

    Ansible Controller
           |
           |
          WinRM
           |
           v
    Windows Server

Ansible Inventory
------------------

Inventory tells Ansible:

     Which servers should I manage?

      [web_servers]
      web01 ansible_host=10.0.2.10
      web02 ansible_host=10.0.2.11
      web03 ansible_host=10.0.2.12

You can then target the entire group:

    ansible web_servers -m ping


What is an Ansible Module?
----------------------------

A module performs a specific operation.


    ping
    copy
    file
    service
    package
    template
    user
    command
    shell


Ad-hoc Command vs Playbook
---------------------------


Ad-hoc command: Used for quick operations

    ansible web_servers -m command -a "uptime"

Good for:

     Quick check
    Quick troubleshooting
    One-time operation


Playbook: Used for repeatable automation.


    - name: Check application servers
      hosts: web_servers
    
      tasks:
        - name: Check uptime
          ansible.builtin.command: uptime


For your career goal, playbooks and roles will be much more important than memorizing ad-hoc commands.

Installation:
-------------

For our lab, we'll use a Linux machine as the Ansible controller.

First check Python:

      python3 --version
      python3 -m pip --version

if pip not available install it:

      yum install pip -y

Then install Ansible and check version:

      python3 -m pip install ansible
      ansible --version

Create Your First Lab
-----------------------

Create a directory:

     mkdir -p ~/ansible-lab
     cd ~/ansible-lab

Create inventory: vi inventory


For now, use your Linux server's IP:


      [linux_servers]
      server01 ansible_host=<same_server_ip> ansible_user=ec2-user ansible_ssh_private_key_file=/root/ansible-lab/key.pem

      in the above we have used the same ip of ec2 where we installed our Ansible.
      user is ec2-user
      we created one .pem file and pasted the private key init. then we provided 400 permissions to file and we used in the inventory with path.

      so server will ping to itself.

Your First Ansible Command:

      ansible linux_servers -i inventory -m ping

      server will ping to it self.

Ad-hoc CMD:

       ansible linux_servers -i inventory -m command -a "uptime"

Your working setup is:
    
    Ansible Controller
           |
           | SSH
           | ec2-user
           | key.pem
           v
    Amazon Linux 2023
    172.31.79.2
           |
           +-- Python 3.9
