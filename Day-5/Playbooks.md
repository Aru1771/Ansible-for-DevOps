Ansible Day 5 — Playbooks
==========================

This is an important step because your main goal is:

    Deploy applications to Linux/Windows servers and perform real-world server administration and patching.

What is an Ansible Playbook?

Until now, you have been running commands like:

    ansible server01 -m dnf -a "name=nginx state=present" -b

and:

    ansible server01 -m systemd -a "name=nginx state=started" -b

This works, but imagine an application deployment requiring 10–20 steps.

You don't want to manually execute 20 commands.

Instead, put the tasks into a YAML file called an Ansible playbook.

    Ansible Controller
           |
           v
       Playbook
           |
           +-- Install package
           +-- Create directory
           +-- Copy application
           +-- Copy configuration
           +-- Start service
           +-- Health check
           |
           v
     Linux Servers


Playbook Structure

A basic playbook looks like:


    ---
    - name: Configure application server
      hosts: server01
      become: true
    
      tasks:
    
        - name: Install nginx
          ansible.builtin.dnf:
            name: nginx
            state: present
    
        - name: Start nginx
          ansible.builtin.systemd:
            name: nginx
            state: started

  There are four important parts here:

    name
     ↓
    What this play is doing
    
    hosts
     ↓
    Which servers
    
    become
     ↓
    Privilege escalation
    
    tasks
     ↓
    What Ansible should do

Understanding YAML

Ansible playbooks use YAML.

YAML is indentation-sensitive.

For example:

    - name: Install nginx
      ansible.builtin.dnf:
        name: nginx
        state: present

Notice:

    - name
      module
        arguments

Don't use random indentation.
This is wrong:

    - name: Install nginx
         ansible.builtin.dnf:

because the indentation is incorrect.

Your First Playbook

Let's create your first playbook.

On the Ansible controller:

    cd /root/ansible-lab

Create:

    vi day5.yml

Put this inside:

    ---
    - name: Install and start nginx
      hosts: server01
      become: true
    
      tasks:
    
        - name: Install nginx
          ansible.builtin.dnf:
            name: nginx
            state: present
    
        - name: Start nginx
          ansible.builtin.systemd:
            name: nginx
            state: started

Save the file.

Understand the Playbook

Let's break it down.

    Play name
    - name: Install and start nginx

This is a description.

It makes the playbook output easier to understand.


hosts

    hosts: server01

This means:
  
    Run this play against server01.

You can also use an inventory group:

    hosts: webservers

Then every host inside webservers is targeted.

become

    become: true

This tells Ansible to use privilege escalation.

Equivalent conceptually to:

    ec2-user
       ↓
    sudo
       ↓
    root

This is necessary for operations such as package installation and service management.


tasks

    tasks:

This contains the actions Ansible should perform.


Task 1

    - name: Install nginx
      ansible.builtin.dnf:
        name: nginx
        state: present

This means:

    Make sure Nginx is installed.

Notice that we aren't doing:

    dnf install nginx -y

We're declaring the desired state:

    nginx → present

Task 2

    - name: Start nginx
      ansible.builtin.systemd:
        name: nginx
        state: started

This means:

    Make sure the Nginx service is running.

Why ansible.builtin.dnf?

Earlier we used:

    dnf:

You may also see:

    ansible.builtin.dnf:

ansible.builtin identifies the module as part of Ansible's built-in collection.

For example:

    ansible.builtin.copy:
    ansible.builtin.file:
    ansible.builtin.systemd:
    ansible.builtin.dnf:
    ansible.builtin.command:

For your learning, we'll use the fully qualified form because it's explicit and is good modern Ansible practice.


Check the Playbook Syntax

Before running a playbook, validate it.

Run:
  
    ansible-playbook --syntax-check day5.yml

Expected:

    playbook: day5.yml

This checks whether the YAML/playbook structure is valid.

It does not execute the tasks.

Run the Playbook

Now:

    ansible-playbook -i inventory day5.yml

You'll see something similar to:

    PLAY [Install and start nginx]
    
    TASK [Install nginx]
    ok
    
    TASK [Start nginx]
    ok
    
    PLAY RECAP
    server01    : ok=2    changed=0    unreachable=0    failed=0

If Nginx wasn't installed before, you may see:

    changed=2

or possibly one changed task depending on the existing service state.


Understanding ok vs changed
----------------------------

This is extremely important.

ok
Means:

    The desired state is already correct.

Example:

    TASK [Install nginx]
    ok

Nginx was already installed.

changed
Means:

    Ansible had to make a change.

Example:

    TASK [Install nginx]
    changed

Nginx wasn't installed, so Ansible installed it.

failed
The task failed.
failed=1

unreachable
Ansible couldn't connect to the server.
unreachable=1

You already experienced this during Day 3.


Run the Playbook Again
--------------------------

This is where you see idempotency.

Run:

    ansible-playbook -i inventory day5.yml

again.
You should see mostly:
ok

rather than:
changed

because the desired state is already satisfied.

    First run
       ↓
    Install nginx
       ↓
    changed
    
    Second run
       ↓
    Nginx already installed
       ↓
    ok

This is exactly why Ansible is useful for DevOps automation.


Add Directory Creation
------------------------

Now let's make this slightly more realistic.
Modify your playbook:

    ---
    - name: Configure application server
      hosts: server01
      become: true
    
      tasks:
    
        - name: Create application directory
          ansible.builtin.file:
            path: /opt/myapp
            state: directory
            mode: '0755'
    
        - name: Install nginx
          ansible.builtin.dnf:
            name: nginx
            state: present
    
        - name: Start nginx
          ansible.builtin.systemd:
            name: nginx
            state: started

Now the playbook does:

    Create /opt/myapp
            ↓
    Install nginx
            ↓
    Start nginx


Add a Configuration File
------------------------

Create a file on the controller:

    echo "Application managed by Ansible" > /root/ansible-lab/app.conf

Now add this task:

    - name: Copy application configuration
      ansible.builtin.copy:
        src: /root/ansible-lab/app.conf
        dest: /opt/myapp/app.conf
        mode: '0644'

Your playbook becomes:

    ---
    - name: Configure application server
      hosts: server01
      become: true
    
      tasks:
    
        - name: Create application directory
          ansible.builtin.file:
            path: /opt/myapp
            state: directory
            mode: '0755'
    
        - name: Install nginx
          ansible.builtin.dnf:
            name: nginx
            state: present
    
        - name: Copy application configuration
          ansible.builtin.copy:
            src: /root/ansible-lab/app.conf
            dest: /opt/myapp/app.conf
            mode: '0644'
    
        - name: Start nginx
          ansible.builtin.systemd:
            name: nginx
            state: started

Now we're getting much closer to a real deployment.


Verify the Deployment
----------------------

Run:

    ansible-playbook -i inventory day5.yml

Then:

    ansible server01 -i inventory -m command \
      -a "ls -l /opt/myapp"

And:

    ansible server01 -i inventory -m command \
      -a "cat /opt/myapp/app.conf"

Expected:

    Application managed by Ansible

The Real-World Pattern

What you're doing now is the foundation for your future application deployment playbooks.


    Playbook
       |
       +-- Prepare server
       |      |
       |      +-- Create directories
       |
       +-- Install dependencies
       |      |
       |      +-- Java
       |      +-- Python
       |      +-- Nginx
       |
       +-- Deploy application
       |      |
       |      +-- Copy artifact
       |      +-- Copy configuration
       |
       +-- Configure service
       |      |
       |      +-- systemd
       |
       +-- Start/restart application
       |
       +-- Health check

This is exactly the direction we'll take your Ansible learning.



Why Playbooks Are Better Than Individual Commands

Imagine production has:
    
    app01
    app02
    app03
    app04
    app05

Without a playbook, you might execute many commands manually.

With:
  
    hosts: app_servers

Ansible can apply the same configuration to the whole group.

              Playbook
                 |
        +--------+--------+
        |        |        |
      app01    app02    app03
        |        |        |
        +--------+--------+
             same
          automation

That's the real power of Ansible.

Day 5 Practical Tasks
------------------------

Task 1

Create:

    day5.yml

with the basic Nginx playbook.

Task 2

Run:
    
    ansible-playbook --syntax-check day5.yml

Task 3

Run:

    ansible-playbook -i inventory day5.yml

Task 4

Run the playbook a second time and observe ok vs changed.

Task 5

Add:
    
    /opt/myapp

using the file module.


Task 6

Create:

app.conf

on the controller and copy it to:
/opt/myapp/app.conf

Task 7

Verify the file from Ansible:

    ansible server01 -i inventory -m command \
      -a "cat /opt/myapp/app.conf"


Task-Play-Book:

        
        ---
        
        - name: installing the nginx
          hosts: server01
          become: true
        
          tasks:
        
            - name: install nginx
              ansible.builtin.dnf:
                name: nginx
                state: present
        
            - name: start nginx
              ansible.builtin.systemd:
                name: nginx
                state: started
        
            - name: create a dir
              ansible.builtin.file:
                path: /root/ansible-lab/file1.txt
                state: directory
                mode: 0777
        
            - name: copy to root
              ansible.builtin.copy:
                src: /root/ansible-lab/app.conf
                dest: /root/
                mode: 0400
        
