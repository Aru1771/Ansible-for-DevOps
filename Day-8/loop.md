Ansible Loops:
==============

Topic 1: Basic loop

Theory: What is a loop?
-----------------------
A loop allows Ansible to execute the same task multiple times using different values.

Without a loop:

Suppose you want to install Git, Tree and Nginx on an EC2 instance.

You might write three separate tasks:

    tasks:
      - name: Install Git
        ansible.builtin.package:
          name: git
          state: present
    
      - name: Install Tree
        ansible.builtin.package:
          name: tree
          state: present
    
      - name: Install Nginx
        ansible.builtin.package:
          name: nginx
          state: present

This works, but it creates unnecessary repetition.

With a loop:

You can write one task and provide a list of packages.

    vars:
      packages:
        - git
        - tree
        - nginx
    
    tasks:
      - name: Install packages
        ansible.builtin.package:
          name: "{{ item }}"
          state: present
        loop: "{{ packages }}"

Much simpler!

Understand item:
----------------

This is the most important part:

name: "{{ item }}"


item is a special variable Ansible provides for the current loop iteration.

Suppose your list is:

    packages:
      - git
      - tree
      - nginx

  <img width="855" height="481" alt="image" src="https://github.com/user-attachments/assets/a278c69d-4758-429f-9ae5-b56029c57359" />

Practical task: Install multiple packages on EC2:
-------------------------------------------------

Create a file named loops.yml.


    ---
    - name: Install packages using Ansible loops
      hosts: dev
      become: true
    
      vars:
        packages:
          - git
          - curl
          - unzip
          - tree
          - nginx
    
      tasks:
        - name: Install required packages
          ansible.builtin.package:
            name: "{{ item }}"
            state: present
          loop: "{{ packages }}"


Topic 2: Dictionary Loops
-------------------------

Theory: What is a dictionary?
------------------------------
A dictionary stores data in key-value pairs.

    Example:
    applications:
      Empower:
        version: FR7
        port: 8080
    
      Payment:
        version: FR2
        port: 9090


Here:

    - Empower and Payment are keys.
    - version and port are their associated values.
In a normal list loop, you use item.

In a dictionary loop, you can use item.key and item.value.


Practical example
Create dictionary-loop.yml:


    ---
    - name: Display application configurations
      hosts: dev
    
      vars:
        applications:
          Empower:
            version: FR7
            port: 8080
    
          Payment:
            version: FR2
            port: 9090
    
      tasks:
        - name: Display application details
          ansible.builtin.debug:
            msg: "Application={{ item.key }}, Version={{ item.value.version }}, Port={{ item.value.port }}"
          loop: "{{ applications | dict2items }}"


Understand dict2items
----------------------

This is important:

    loop: "{{ applications | dict2items }}"

dict2items converts a dictionary into a list of key-value objects so Ansible can iterate over it.
