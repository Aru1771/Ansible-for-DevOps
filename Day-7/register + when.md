Next, we'll learn how to use register + when together. This is particularly useful for production troubleshooting and deployment automation.


Topic 2: register + when:
-------------------------

<img width="888" height="333" alt="image" src="https://github.com/user-attachments/assets/121f6e54-b57b-4041-96c1-331a93f5542b" />

Practical example:
-------------------

    ---
    - name: Check Nginx installation
      hosts: dev
      become: true
    
      tasks:
        - name: Check whether Nginx is installed
          ansible.builtin.command: rpm -q nginx
          register: nginx_status
          failed_when: false
          changed_when: false
    
        - name: Display installation confirmation
          ansible.builtin.debug:
            msg: "Nginx is already installed"
          when: nginx_status.rc == 0
    
        - name: Install Nginx if missing
          ansible.builtin.dnf:
            name: nginx
            state: present
          when: nginx_status.rc != 0

Understand line by line:
-----------------------

First task:

    register: nginx_status

* Stores the command result in a variable called nginx_status.

* The registered result includes fields such as:

      - nginx_status.rc — command return code.
      - nginx_status.stdout — standard output.
      - nginx_status.stderr — error output.

<img width="716" height="207" alt="image" src="https://github.com/user-attachments/assets/ef4f0657-771f-4fc6-925f-29b51b9e8f4b" />

<img width="1012" height="688" alt="image" src="https://github.com/user-attachments/assets/5e2ddeb3-4726-4128-a72e-5f8fada596d1" />

