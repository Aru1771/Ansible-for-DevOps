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
-------------------------

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

Variable Types:
----------------
Ansible variables aren't limited to strings.

    String: app_name: myapp
    
    Number: app_port: 8080
    
    List: packages:
          - nginx
          - git
          - curl
    
    Dictionary:application:
                  name: myapp
                  port: 8080
                  environment: production


We will use these heavily later.



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

Multiple Variables:
---------------------
You can define multiple variables:


    vars:
      package_name: nginx
      package_state: present
      service_name: nginx
      web_port: 80


Then:

    tasks:
    
      - name: Install package
        ansible.builtin.package:
          name: "{{ package_name }}"
          state: "{{ package_state }}"
    
      - name: Start service
        ansible.builtin.service:
          name: "{{ service_name }}"
          state: started
          enabled: true




Using a List Variable
----------------------

    ---
    - name: Install packages
      hosts: all
      become: true
    
      vars:
        packages:
          - nginx
          - git
          - curl
    
      tasks:
    
        - name: Install packages
          ansible.builtin.package:
            name: "{{ item }}"
            state: present
          loop: "{{ packages }}"
    
Flow:
    
    packages
       |
       +── nginx
       +── git
       +── curl
              |
              ▼
            loop
              |
              ▼
          package module  
        
    

Dictionary Variables

Suppose:

    application:
      name: payment-api
      port: 8080
      environment: production

You can access individual values:

    {{ application.name }}

    {{ application.port }}

    {{ application.environment }}

Example:

    - name: Show application
      ansible.builtin.debug:
        msg: "Application {{ application.name }} runs on port {{ application.port }}"

Output conceptually:

    Application payment-api runs on port 8080








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


Real DevOps Use Case — Software Installation
---------------------------------------------

    ---
    - name: Install software using variables
      hosts: all
      become: true
    
      vars:
        package_name: nginx
    
      tasks:
    
        - name: Install software
          ansible.builtin.package:
            name: "{{ package_name }}"
            state: present
    
The important relationship is:

    package_name
         │
         ▼
    "nginx"
         │
         ▼
    package module
         │
         ▼
    Install nginx

Tomorrow you could change:

    package_name: nginx

to:

    package_name: git

without changing the task.    


