SSH Connectivity with Linux
============================

Today we'll understand how Ansible connects to Linux servers using SSH, and more importantly, how to troubleshoot SSH problems in a real DevOps environment.

Why Does Ansible Use SSH?
---------------------------


Ansible needs a way to execute operations on the managed server.

For Linux, the default mechanism is SSH.

For example:

    ssh ec2-user@172.31.79.2

Ansible essentially automates this communication.

Instead of manually doing:

    ssh ec2-user@server01

and then:

    uptime

you can run:

    ansible server01 -m command -a "uptime"

Ansible handles the connection and execution for you.

SSH Authentication
-------------------
There are two common authentication methods:

Password authentication

    Controller
        |
        | username + password
        v
    Server
    
SSH key authentication ⭐
--------------------------
    
    Controller
        |
        | private key
        v
    Server

For AWS EC2, SSH key authentication is commonly used.

Your setup:
    
    Private key:
     /root/ansible-lab/key.pem

    Remote user:
     ec2-user

Public Key vs Private Key
---------------------------

This is extremely important.

SSH uses a key pair:

             SSH Key Pair
                  |
          +-------+-------+
          |               |
       Private          Public
        Key              Key
       key.pem         authorized_keys
          |               |
      Controller        Server

Private key

Your controller has:

    key.pem

You should never share the private key.

Public key

The server has the corresponding public key in:

      ~/.ssh/authorized_keys

The server uses that public key to verify the private key presented by the client.


Why Did You Get Permission denied?
------------------------------------

Earlier you had:

    root@172.31.79.2:
    Permission denied (publickey)

The problem was:
-----------------

      Ansible
         |
         | tried SSH as root
         v
      EC2 Server
         |
         X
      root authentication rejected


We changed:

    ansible_user=ec2-user

and provided:

    ansible_ssh_private_key_file=/root/ansible-lab/key.pem

Then:

    Ansible
       |
       | ec2-user + key.pem
       v
    EC2
       |
       ✓


SSH Configuration in Inventory


Your current inventory:

    [dev]
    server01 ansible_host=172.31.79.2 ansible_user=ec2-user ansible_ssh_private_key_file=/root/ansible-lab/key.pem

Let's understand each part.

    ansible_host
    ansible_host=172.31.79.2

This is the actual IP address Ansible connects to.

    ansible_user
    ansible_user=ec2-user

This tells Ansible:

Log into the server as ec2-user.

    ansible_ssh_private_key_file
    ansible_ssh_private_key_file=/root/ansible-lab/key.pem

This tells Ansible:

Use this private key for SSH authentication.



Test SSH Outside Ansible
---------------------------

This is a very important DevOps troubleshooting technique.

Before troubleshooting Ansible, test SSH manually:

    ssh -i /root/ansible-lab/key.pem ec2-user@172.31.79.2

If this works:

    [ec2-user@ip-172-31-79-2 ~]$

then basic SSH connectivity is working.

If it doesn't work, don't troubleshoot Ansible yet.

Fix SSH first.


SSH Troubleshooting Flow
-------------------------

When Ansible says:

UNREACHABLE

don't immediately change the playbook.

Use this flow:

    Ansible UNREACHABLE
           |
           v
    Can I ping/reach the IP?
           |
           v
    Can I SSH manually?
           |
       +---+---+
       |       |
      No      Yes
       |       |
    Fix SSH   Check
              Ansible



Check Port 22
---------------

SSH normally uses:

    TCP 22

From the controller:

    nc -vz 172.31.79.2 22

Successful output may look like:

    Connection to 172.31.79.2 22 port [tcp/ssh] succeeded!

If you get:

    Connection timed out

think about:

    Security Group
    Network ACL
    Routing
    Firewall
    Network connectivity


Check SSH Service
-------------------
On the managed server:

    sudo systemctl status sshd

You should see something like:

    Active: active (running)

If it isn't running:

    sudo systemctl start sshd

To make it start automatically:

    sudo systemctl enable sshd


Check Private Key Permissions: chmod 400 /root/ansible-lab/key.pem

Verbose Mode: add -v /-vv /-vvv to troubleshoot the issue.

