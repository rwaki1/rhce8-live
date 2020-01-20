# rhce8-live

## Ansible Lab with Vagrant
To prepare a sample lab environment for testing Ansible , you can use Vagrant to create an environment with 3 nodes as follows:
```bash
lab-ans-control :  192.168.4.200  control.example.com
lab-ans-ansible1:  192.168.4.201  ansible1.example.com
lab-ans-ansible2:  192.168.4.202  ansible2.example.com
```

---
 **NOTE**

 * The environment is not secure and it is only for testing Ansible.
 * All passwords of _ansible_ users are **ansible**.
 * The _vagrant_ user is sudoer on all nodces.
 * The _ansible_ user is sudoer in all of nodes.
 * In the home of _ansible_ user on lab-ans-control, the [sandervanvugt/rhce8-live](https://github.com/sandervanvugt/rhce8-live.git) repository is already checked out here: `/home/_ansible_/rhce8-live`.
---

Steps to prepare the environment on your host Linux:

* Install VirtualBox&copy; based on your Linux distribution from [here](https://www.virtualbox.org/wiki/Linux_Downloads)
* Install Vagrant&copy; based on your Linux distribution from [here](https://www.vagrantup.com/docs/installation/)
* Install git
* Running the Vagrant environment on your host Linux:
    * To check, if VirtualBox is installed correct
        ```bash
        $ vboxmanage --version
        5.2.36r135684       # Without any Error
        ```
    * To check, if Vagrant is installed correct
        ```bash
        $ vagrant --version
        Vagrant 2.x.x       # Without any Error
        ```

    * Clone repository of the course into your existing path like /path/to/repos
        ```bash
        $ git clone https://github.com/sandervanvugt/rhce8-live.git /path/to/repos/rhce8-live
        $ cd /path/to/repos/rhce8-live/ansible-lab
        ```

    * Check the validity of Vagrantfile with your installed version
        ```bash
        $ vagrant validate
        Vagrantfile validated successfully.
        ```

    * Start Vagrant
        ```bash
        $ vagrant up --provider=virtualbox
        # It can take a long time depending on the speed of your PC and internet access.
        # You should see a lot of log for each node.
        ```

    * Vagrant Status
        ```bash
        $ vagrant status                
        Current machine states:

        lab-ans-ansible1          running (virtualbox)
        lab-ans-ansible2          running (virtualbox)
        lab-ans-control           running (virtualbox)
        ```

    * Access to Vagrant Nodes via SSH:
        ```bash
        $ vagrant ssh lab-ans-control   
        Last login: Mon Jan 20 11:26:32 2020 from 10.0.2.2
        [vagrant@control ~]$ su - ansible
        Password: ansible
        Last login: Mon Jan 20 11:26:41 UTC 2020 on pts/0

        # Check the access to managed node 1
        [ansible@control ~]$ ssh ansible1.example.com
        Last login: Mon Jan 20 11:26:52 2020 from 192.168.4.200
        [ansible@ansible1 ~]$ logout
        Connection to ansible1.example.com closed.     

        # Check the access to managed node 2
        [ansible@control ~]$ ssh ansible2.example.com
        Last login: Mon Jan 20 11:26:57 2020 from 192.168.4.200
        [ansible@ansible2 ~]$ logout
        Connection to ansible2.example.com closed.
        ```
        ```bash
        $ vagrant ssh lab-ans-ansible1
        ```
        ```bash
        $ vagrant ssh lab-ans-ansible2
        ```

    * Access to all created nodes via GUI of VirtualBox is possible,  more info [here](https://www.virtualbox.org/manual/ch01.html)
        ```bash
        $ vagrant ssh lab-ans-control
        [vagrant@control ~]$ sudo su - ansible
        Last login: Mon Jan 20 12:33:58 UTC 2020

        [ansible@control ~]$ ll
        total 0
        drwxrwxr-x. 14 ansible ansible 244 Jan 20 12:34 rhce8-live

        [ansible@control ~]$ ll rhce8-live/
        total 20
        -rwxrwxr-x. 1 ansible ansible  151 Jan 20 12:34 countdown
        drwxrwxr-x. 4 ansible ansible  212 Jan 20 12:34 lesson11
        drwxrwxr-x. 3 ansible ansible  156 Jan 20 12:34 lesson12
        drwxrwxr-x. 3 ansible ansible  119 Jan 20 12:34 lesson13
        drwxrwxr-x. 3 ansible ansible  271 Jan 20 12:34 lesson15
        drwxrwxr-x. 7 ansible ansible 4096 Jan 20 12:34 lesson16
        drwxrwxr-x. 2 ansible ansible  102 Jan 20 12:34 lesson2
        drwxrwxr-x. 2 ansible ansible  181 Jan 20 12:34 lesson4
        drwxrwxr-x. 4 ansible ansible   90 Jan 20 12:34 lesson5
        drwxrwxr-x. 4 ansible ansible   32 Jan 20 12:34 lesson7
        drwxrwxr-x. 2 ansible ansible 4096 Jan 20 12:34 lesson8
        drwxrwxr-x. 3 ansible ansible  194 Jan 20 12:34 lesson9
        -rw-rw-r--. 1 ansible ansible   13 Jan 20 12:34 README.md
        -rw-rw-r--. 1 ansible ansible  457 Jan 20 12:34 setup_repo.yml
        ```
    * To stop the nodes of Vagrant `vagrant halt`
    * To delete the nodes of Vagrant `vagrant destroy`
    * Running a sample playbook _**rhce8-live/lesson4/install-httpd.yml**_ :
        * The sample on the control node:
            ```bash
            vagrant ssh lab-ans-control
            [vagrant@control ~]$ sudo su - ansible
            Last login: Mon Jan 20 12:51:13 UTC 2020
            [ansible@control ~]$ cd rhce8-live/lesson4
            [ansible@control lesson4]$ cat install-httpd.yml
            ---
            - name: install httpd
            hosts: ansible2.example.com
            tasks:
            - name: install package
            package:
              name: httpd
              state: present
            - name: create an index.html
            copy:
              content: 'welcome to this webserver'
              dest: /var/www/html/index.html
            - name: start the service
            service:
              name: httpd
              state: started
              enabled: true
            - name: open firewall
            firewalld:
              service: http
              permanent: yes
              state: enabled
             ```
        * The execution result of _ansible-playbook_ on the control node:
            ```bash
            [ansible@control lesson4]$ ansible-playbook install-httpd.yml

            PLAY [install httpd] **************************************************************************************************

            TASK [Gathering Facts] ************************************************************************************************
            ok: [ansible2.example.com]

            TASK [install package] ************************************************************************************************
            changed: [ansible2.example.com]

            TASK [create an index.html] *******************************************************************************************
            changed: [ansible2.example.com]

            TASK [start the service] **********************************************************************************************
            changed: [ansible2.example.com]

            TASK [open firewall] **************************************************************************************************
            changed: [ansible2.example.com]

            PLAY RECAP ************************************************************************************************************
            ansible2.example.com       : ok=5    changed=4    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
            ```
        * Check the installed httpd on the control node:
            ```bash
            [ansible@control lesson4]$ curl http://ansible2.example.com
            welcome to this webserver
            ```        
        * Check the installed httpd on the second managed node, the same result:
            ```bash
            vagrant ssh lab-ans-ansible2
            [vagrant@ansible2 ~]$ curl http://localhost/
            welcome to this webserver
            ```
