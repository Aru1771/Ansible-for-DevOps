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

<img width="997" height="806" alt="image" src="https://github.com/user-attachments/assets/f1c844d8-6679-467b-bb20-6a61b6c960e2" />

<img width="1041" height="325" alt="image" src="https://github.com/user-attachments/assets/61e36e52-e586-4bb3-a553-f99e71d207fa" />

register + when — Practical Exercise

<img width="1002" height="427" alt="image" src="https://github.com/user-attachments/assets/e35629d5-99e6-49f8-a971-42772fc07fd4" />

<img width="1003" height="695" alt="image" src="https://github.com/user-attachments/assets/82c14904-b428-4db0-aee2-6abad55f271a" />

<img width="992" height="388" alt="image" src="https://github.com/user-attachments/assets/e957c7b0-c0ed-4527-a71e-2be1725b5d94" />

<img width="975" height="636" alt="image" src="https://github.com/user-attachments/assets/e49f9875-457e-402b-b405-e17cf49077ff" />

<img width="970" height="557" alt="image" src="https://github.com/user-attachments/assets/cab4cbf5-e124-43f7-afab-f698af9cfbf7" />
