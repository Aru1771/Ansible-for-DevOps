Ansible Inventory
=================

Ansible inventory tells Ansible: Which servers do I manage, and how should I connect to them?

Why Do We Need Groups?
------------------------

In a real company, you won't have one server.

For example:

    Production
       |
       +-- web01
       +-- web02
       +-- app01
       +-- app02
       +-- db01

You don't want to write commands individually for every server.

Instead, group them.

    [web_servers]
    web01
    web02
    
    [app_servers]
    app01
    app02
    
    [db_servers]
    db01
Now you can target a complete group.


Your First Real-World Inventory
---------------------------------

Let's modify your current inventory.

Create:

    cd /root/ansible-lab

Then:

    vi inventory

For now, because you have only one server, we'll use the same server in multiple conceptual groups only for learning.

    [dev]
    server01 ansible_host=172.31.79.2 ansible_user=ec2-user ansible_ssh_private_key_file=/root/ansible-lab/key.pem
    
    [web_servers]
    server01
    
    [linux_servers]
    server01

Notice something important.

We define the connection details only once:

    [dev]
    server01 ansible_host=172.31.79.2 ansible_user=ec2-user ansible_ssh_private_key_file=/root/ansible-lab/key.pem
    
    Then:
    
    [web_servers]
    server01
    
    and:
    
    [linux_servers]
    server01

refer to the same host.
Now you can target a complete group.


Test the Groups
---------------
Test dev
      
      ansible dev -i inventory -m ping

Test web_servers

    ansible web_servers -i inventory -m ping

Test linux_servers

    ansible linux_servers -i inventory -m ping

All three should return:

SUCCESS
"ping": "pong"


Important Inventory Concept
----------------------------
The same server can belong to multiple groups.

For example:

                    server01
                       |
          +------------+------------+
          |            |            |
         dev      web_servers   linux_servers

This is very useful.

A server could be:

    Environment: production
    Application: payment
    OS: Linux
    Role: web server


So it could belong to:

    [prod]
    server01
    
    [web]
    server01
    
    [linux]
    server01
    
    [payment]
    server01

Later you'll use this to control which servers get deployed or patched.


Parent Groups — :children
-----------------------------

Now we can create a parent group.

Suppose:

    dev
    stage
    prod

are all Linux environments.

We can create:

    [linux_servers:children]
    dev
    stage
    prod

Complete example:

    [dev]
    dev-web01
    dev-app01
    
    [stage]
    stage-web01
    stage-app01
    
    [prod]
    prod-web01
    prod-web02
    prod-app01
    prod-app02
    
    [linux_servers:children]
    dev
    stage
    prod

Now:

    ansible linux_servers -m ping

targets:

    dev-web01
    dev-app01
    stage-web01
    stage-app01
    prod-web01
    prod-web02
    prod-app01
    prod-app02


Why :children Is Useful
---------------------------

Imagine you need to patch all Linux servers.

Instead of:

    ansible dev -m ...
    ansible stage -m ...
    ansible prod -m ...

you can use:

    ansible linux_servers -m ...

This becomes extremely useful for our future:

Linux patching

    linux_servers
          |
          +-- dev
          +-- stage
          +-- prod
Windows patching

    windows_servers
          |
          +-- dev
          +-- stage
          +-- prod

We'll eventually build something like:

                 servers
                    |
          +---------+---------+
          |                   |
        Linux               Windows
          |                   |
     +----+----+         +----+----+
     |    |    |         |    |    |
    dev stage prod      dev stage prod

Host Variables
---------------
You can define variables for a specific server.

Example:

    [web_servers]
    server01 ansible_host=172.31.79.2 ansible_user=ec2-user

You can add:

    server01 ansible_port=22

For example:

    server01 ansible_host=172.31.79.2 ansible_user=ec2-user ansible_port=22

These are host variables.

Group Variables
------------------
Suppose all production servers use the same SSH user.

Instead of repeating:

ansible_user=ec2-user

on every server, we can use group variables.

Example:

ansible-lab/
├── inventory
└── group_vars/
    └── prod.yml

prod.yml:

    ansible_user: ec2-user
    ansible_port: 22

Then every host in prod gets these values.

This becomes much cleaner when you have dozens or hundreds of servers.

Your Day 2 Practice
-------------------
We'll keep today's practice focused.

Task 1 — Create these groups

Modify your inventory to contain:

    [dev]
    server01 ansible_host=172.31.79.2 ansible_user=ec2-user ansible_ssh_private_key_file=/root/ansible-lab/key.pem
    
    [web_servers]
    server01
    
    [linux_servers]
    server01
Task 2 — Test each group

Run:

    ansible dev -i inventory -m ping

Then:

    ansible web_servers -i inventory -m ping

Then:

      ansible linux_servers -i inventory -m ping

Task 3 — Create a parent group

Add:

    [linux:children]
    dev

Then run:

    ansible linux -i inventory -m ping

You should see server01.

Task 4 — Test the inventory

Run:

    ansible-inventory -i inventory --graph

You should see something similar to:

    @all:
      |--@dev:
      |    |--server01
      |--@linux:
      |    |--@dev:
      |         |--server01
      |--@linux_servers:
      |    |--server01
      |--@web_servers:
           |--server01

This command is very useful in real-world troubleshooting because it lets you see how Ansible interprets your inventory.
