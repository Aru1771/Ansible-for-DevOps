Ansible Modules
=================

We'll understand how modules are used for real-world application deployment and server administration.

What is an Ansible Module?
--------------------------

An Ansible module is a piece of functionality that Ansible uses to perform a specific task on a managed server.

    Ansible
       |
       +-- ping module
       +-- command module
       +-- shell module
       +-- package module
       +-- service module
       +-- copy module
       +-- file module
       +-- user module
       +-- systemd module

Each module has a specific purpose.

Why Do We Need Modules?
------------------------

Suppose you want to install Nginx.

You could manually SSH to the server:

    ssh ec2-user@server

then:

    sudo dnf install nginx


With Ansible, you can automate this.

For Amazon Linux 2023:

    ansible server01 -m dnf -a "name=nginx state=present" -b

Here:

    -m dnf

means:

    Use the dnf module.

And:

    name=nginx state=present

means:

    Make sure Nginx is installed.

This is much more powerful than simply running shell commands.


Module Syntax

The general syntax is:

    ansible <host/group> -m <module> -a "<arguments>"

Example:

    ansible server01 -m ping

Another:

    ansible server01 -m command -a "hostname"

Break it down:

    ansible
       |
       +-- server01      → target
       |
       +-- -m command    → module
       |
       +-- -a "hostname" → module arguments

ping Module:
-------------

The Ansible ping module does not work like normal network ICMP ping.

It actually verifies that:

      Ansible can connect
      Python is available
      An Ansible module can execute

command Module
---------------

This executes a command on the remote server.

Examples:

    ansible server01 -m command -a "hostname"

command does not process shell features.

For example, this isn't appropriate:

      ansible server01 -m command -a "cat /var/log/messages | grep error"

because | is a shell feature.

For that, you may use the shell module.


shell Module
-------------

The shell module executes commands through the remote shell.

Example:

    ansible server01 -m shell -a "cat /var/log/messages | grep error"

You can use shell operators such as:

    |
    >
    >>
    &&
    ||
    *

For example:

      ansible server01 -m shell -a "df -h | grep /"

But don't overuse shell

A common beginner mistake is:

Everything → shell

That's not good Ansible practice.

If Ansible has a dedicated module for the operation, generally prefer the dedicated module.

For example:

❌:

    ansible server01 -m shell -a "dnf install nginx -y"

Prefer:

    ansible server01 -m dnf -a "name=nginx state=present" -b

Why?

Because the dnf module understands the desired state.


command vs shell
-----------------
Very important interview question.

| `command`                 | `shell`                     |               |
| ------------------------- | --------------------------- | ------------- |
| Executes command          | Executes through shell      |               |
| Safer for simple commands | Useful for shell operations |               |
| No shell operators        | Supports shell operators    |               |
| No `                      | `, `>`, `&&` processing     | Supports them |


Example:

Command

    ansible server01 -m command -a "uptime"

Shell

    ansible server01 -m shell -a "df -h | grep /"

Think:

    Simple command → command
    
    Shell functionality → shell
    

become — Very Important
-----------------------

Your current SSH user is:

    ec2-user

But ec2-user normally doesn't directly perform every administrative operation.

For example:

    dnf install nginx

requires elevated privileges.

Ansible can use sudo through become.

Example:

      ansible server01 -m dnf -a "name=nginx state=present" -b

-b means:

    become

In simple terms:

Run the task with elevated privileges.

Flow:

    Ansible
       |
       | SSH
       v
    ec2-user
       |
       | sudo / become
       v
    root privileges
       |
       v
    Install package

This concept will become extremely important for server patching.


dnf Module
-----------

Because you're using Amazon Linux 2023, you'll commonly work with dnf.

Check whether Nginx is installed:

    ansible server01 -m dnf -a "name=nginx state=present" -b

If it isn't installed:

    changed: true

If it's already installed:

    changed: false

This is idempotency.


What Is Idempotency?
------------------------

This is one of the most important Ansible concepts.

Suppose you run:

      ansible server01 -m dnf -a "name=nginx state=present" -b

First time:

    changed: true

Nginx gets installed.

Run the exact same command again:

    ansible server01 -m dnf -a "name=nginx state=present" -b

Ideally:

    changed: false

because the desired state is already achieved.

That's idempotency.

    Desired state:
    Nginx = installed
    
    Current:
    Nginx = not installed
    
            ↓
    
    Ansible installs it
    
            ↓
    
    Nginx = installed
    
            ↓
    
    Run again
    
            ↓
    
    No change required

This is one of the major advantages of Ansible.


service / systemd Module
--------------------------
Now suppose Nginx is installed.

We want it running.

You can use:

      ansible server01 -m service -a "name=nginx state=started" -b

We want it running.

You can use:

      ansible server01 -m systemd -a "name=nginx state=started" -b
For modern Linux systems, you'll often encounter systemd.

**state=started** only turns the service on right now for the current session. If the server reboots, the service will remain off unless it was previously enabled.

Enable a Service at Boot
-------------------------

Starting a service means:

Start it now.

Enabling means:

Start it automatically after reboot.

Example:

    ansible server01 -m systemd -a "name=nginx state=started enabled=yes" -b

Now:

    state=started
          ↓
    Run now

    enabled=yes
          ↓
    Start after reboot

• enabled=yes tells the operating system to create the necessary boot links so the service starts automatically every time the server boots up.


copy Module
------------
The copy module transfers files from the Ansible controller to the managed server.

Suppose on the controller you have:

    /root/ansible-lab/index.html

You can copy it:

    ansible server01 -m copy \
      -a "src=/root/ansible-lab/index.html dest=/tmp/index.html"

Flow:

    Ansible Controller
            |
            | copy
            v
    Managed Server
            |
            v
    /tmp/index.html

This will become useful for application configuration files.

* dest always refers to the file path on the managed remote server (server01).
* src always refers to the file path on the control node (your machine running Ansible),

file Module
-----------
The file module manages:

    files
    directories
    permissions
    ownership
    symbolic links

Create a directory:

    ansible server01 -m file \
      -a "path=/opt/myapp state=directory" -b

Now:

    /opt/myapp

exists.


Manage Permissions
--------------------

For example:

    ansible server01 -m file \
      -a "path=/opt/myapp mode=0755" -b

You can also manage ownership:

    ansible server01 -m file \
      -a "path=/opt/myapp owner=ec2-user group=ec2-user" -b

This is useful when deploying applications.

user Module
-------------

You can manage Linux users.

Create a user:

    ansible server01 -m user \
      -a "name=appuser state=present" -b

Remove a user:

    ansible server01 -m user \
      -a "name=appuser state=absent" -b

Again, notice the desired state:

    state=present
    state=absent

Modules You Should Know for Your Career
---------------------------------------

For your goal of application deployment + Linux/Windows patching, these are particularly important:


      Linux
       |
       +-- command
       +-- shell
       +-- dnf
       +-- yum
       +-- package
       +-- service
       +-- systemd
       +-- copy
       +-- file
       +-- template
       +-- user
       +-- group
       +-- lineinfile
       +-- get_url

Later, for Windows:

    Windows
     |
     +-- win_shell
     +-- win_command
     +-- win_service
     +-- win_copy
     +-- win_package
     +-- win_user
     +-- win_updates

* win_updates will be particularly important for your Windows patching goal.

We will get there later in the learning path.



18. Real-World Application Deployment

Eventually you'll do something like:

      Developer
          |
          v
      Jenkins
          |
          v
      Build application
          |
          v
      Create artifact
          |
          v
      Ansible
          |
          +-- Create application directory
          |
          +-- Copy artifact
          |
          +-- Copy configuration
          |
          +-- Set permissions
          |
          +-- Restart service
          |
          +-- Health check
          |
          v
      Application Server

The modules we are learning today are the building blocks for this.
