Today's goal
-------------
By the end of today, you should understand how to:

    - Define variables
    - Use variables inside playbooks
    - Use variables for paths, packages, users, ports, etc.
    - Understand {{ variable_name }} syntax
    - Use variables in tasks
    - Understand why variables are important in real-world deployments
    - Troubleshoot variable-related errors
    - Answer Ansible variable interview questions


Why do we need variables?

Imagine you have this:

    - name: Create application directory
      ansible.builtin.file:
        path: /opt/myapp
        state: directory

Today your application is:

    myapp

Tomorrow it might be:

    payment-service

If you hard-code /opt/myapp everywhere, you have to modify many places.

Instead, define:

    vars:
      app_name: myapp

Then use:

    path: "/opt/{{ app_name }}"

Now if you change:

app_name: payment-service

the playbook automatically works with:

      /opt/payment-service

That's the main purpose of variables.


Basic variable syntax
-------------------------

    vars:
      app_name: myapp

Use it with:

    {{ app_name }}

So:
  
  - name: Create application directory
    ansible.builtin.file:
      path: "/opt/{{ app_name }}"
      state: directory

Remember

Define variable:

    app_name: myapp

Use variable:

    {{ app_name }}

{{ }} is called Jinja2 template syntax.

Your first practical task
-----------------------

Create:

    day6.yml

Use this:

    ---
    - name: Ansible Variables Practice
      hosts: server01
      become: true
    
      vars:
        app_name: myapp
    
      tasks:
    
        - name: Create application directory
          ansible.builtin.file:
            path: "/opt/{{ app_name }}"
            state: directory
            mode: '0755'
    
        - name: Display application name
          ansible.builtin.debug:
            msg: "Application name is {{ app_name }}"

Understand the structure

    vars:
      app_name: myapp

We created a variable.

Then:

    path: "/opt/{{ app_name }}"

Ansible replaces:

    {{ app_name }}

with:

    myapp

So the actual path becomes:

    /opt/myapp

And:

    msg: "Application name is {{ app_name }}"

becomes:

    Application name is myapp

Execute it

First:

    ansible-playbook -i inventory day6.yml --syntax-check

Then:

    ansible-playbook -i inventory day6.yml

Then verify:

    ls -ld /opt/myapp


Multiple variables
------------------

Add these variables:

    vars:
      app_name: git-app
      app_version: "1.0"
      app_port: 8080

Then change your debug task to:

    - name: Display application details
      ansible.builtin.debug:
        msg: "Application {{ app_name }} version {{ app_version }} is running on port {{ app_port }}"

Expected output:

    Application git-app version 1.0 is running on port 8080

This will teach you how multiple variables can be used together in a real application deployment.
