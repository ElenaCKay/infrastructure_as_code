# Infrastucture as code

Where we codify everything, such as user stories. This is implementing the stories into code. Another example, is the bash script we made to install nginx, we want to be able to reuse this possibly for 100s of times. I can be hybrid, local or on the cloud. I am going to be using AWS for this task.

Other ways to use it: Chef, puppet. I am using Ansible.

## What is Ansible?

This is the configuration management tool used heavily in DevOps. It is simple to use. It is agentless, lightweight, doesnt have dependencies. Setting up the controller (master node) and other things.

![ansible overview](imgs/ansible-overview.png)

### Steps to set up ansible with app, db and controller

1. Create 2 EC instances with ubuntu 18.04 and security group SSH ami-0a7493ba2bc35c1e9
2. Name controller, db and app
3. SSH into the controller vm
4. update - `sudo apt update -y`
5. install common software 

`sudo apt-get install software-properties-common`

6. Add a repo for ansible

`sudo apt-add-repository ppa:ansible/ansible`

7. update again `sudo apt update -y`
8.  Install ansible `sudo apt install ansible -y`

![Alt text](imgs/ansible-version-linux.png)

9. SSH into app and db vms and update and upgrade
10. SSH back into controller (agent node)
11. cd into ansible folder `cd /etc/ansible/` to check if ansible is there and available
12. Add tech241.pem file to .ssh folder (copy it from local)
13. Give it correct permissions `sudo chmod 400 tech241.pem` (just read permissions)
14. In the controller vm - ssh into the app vm and the db vm to test the connection and see if it works
15. back in the controller cd into ansible location `cd /etc/ansible/`

![Alt text](<imgs/ls of ansible folder.png>)

16. Install tree `sudo apt install tree -y` for a better view
17. ansible ping command `sudo ansible all -m ping` - doesnt work yet as host file is empty, -m stands for module

![Alt text](imgs/ping-command.png)

18. `sudo nano hosts`

![Alt text](imgs/add-ip-host.png)

Addition in to the hosts file:

    [web]
    ec2-instance ansible_host=54.155.38.93 ansible_user=ubuntu ansible_ssh_private_key_file=~/.ssh/tech241.pem

The ansible_host is the public app ip address.

19.  Run `sudo ansible web -m ping`
    
![Alt text](imgs/ping-working.png)

Remember that in AWS the public ip address changes so you have to go back into the hosts file when you restart the vms and change the ip addresses:

controller ip: 3.252.39.68
app ip: 34.245.13.204
db ip: 34.245.78.113

Before you can do an update you need to find out the operating system of the node as the different OS have different commands. adhoc commands are used for this. So you dont need to ssh into every instance. It needs sudo permissions for ansible. [Link to adhoc command page](https://docs.ansible.com/ansible/latest/command_guide/intro_adhoc.html)

Example:

`sudo ansible web -a "uname -a"` : -a is argument, this will guve us the os of the web vm. It will first ping to chack if it is running, then it will check if it has access using ssh.

![Alt text](imgs/adhoc-command-uname.png)

`sudo ansible web -a "date"` : Used to check where the vm is running as they can be all over the world - important is different regions have different weekends and times.

`sudo ansible web -a "free"` : how much memory is availble. Important if you need to install things and you need to see if there is enough space

`sudo ansible web -a "la -l"` : checks to see if it has the correct files and dependencies

Find an adhoc command to transfer file from controller to agent.

`sudo ansible web -m ansible.builtin.copy -a "src=~/test.txt dest=~"`
`sudo ansible web -a "cat test.txt"`

![Alt text](imgs/adhoc-command-copy-file.png)

End goal for today is to get ansible to install and configure things in the app and db vms. 

For the web vm:

- Install nginx
- Node security group must allow port 80 and port 22 (this is done in the aws portal security group)

### Write a playbook using Yaml

`cd /etc/ansible`
`sudo nano nginx-playbook.yml`

Lots of devops tools use yaml including ansible. Playbooks are reusable and can be used on as many nodes as you would like.
A yml file always starts with 3 dashes ---.

Indentation is key, dont use tab!

![Alt text](imgs/yaml-script-web.png)

`sudo ansible-playbook nginx-playbook.yml` : Runs the playbook
`sudo ansible web -a "systemctl status nginx"` : Checks if nginx is running in the web

Blocker: This will only work if a update and upgrade is carried out first! You can add this in the playbook before installing nginx like so:

```  - name: Update
    command: sudo apt update -y

  - name: Upgrade
    command: sudo apt upgrade -y
```

#### Creating a playbook for node:

![Alt text](imgs/node-playbook.png)

This installs node to the web vm and git clones the app folder to the vm. I still need to work out how to do the npm install and npm start.


#### Playbook with node and npm:

![Alt text](imgs/node-playbook-with-npm-start.png)

This will run the app but because there is no pm2 you cant do anything in the terminal as it runs.

```
---
- hosts: web
  gather_facts: yes
  become: true

  tasks:
    - name: Installing Node.js
      apt:
        pkg: nodejs
        state: present

    - name: Clone GitHub repository
      git:
        repo: https://github.com/ElenaCKay/tech241_aparta_app.git
        dest: /home/ubuntu/app
        version: main

    - name: Installing npm
      apt:
        pkg: npm
        state: present

    - name: Run npm install
      npm:
        path: /home/ubuntu/app/app
        name: install

    - name: Npm Start
      command: sudo npm start
      args:
        chdir: /home/ubuntu/app/app

```

#### Got it working with pm2:

![Alt text](imgs/node-playbook-with-pm2.png)

```---
- hosts: web
  gather_facts: yes
  become: true

  tasks:
    - name: Update APT cache
      apt:
        update_cache: yes

    - name: Upgrade packages
      apt:
        upgrade: dist

    - name: Clone GitHub repository
      git:
        repo: https://github.com/ElenaCKay/tech241_aparta_app.git
        dest: /home/ubuntu/app
        version: main

    - name: Install Node.js
      get_url:
        url: https://deb.nodesource.com/setup_12.x
        dest: /tmp/node_setup.sh

    - name: Run Node.js setup script
      command: sudo -E bash /tmp/node_setup.sh

    - name: Install Node.js package
      apt:
        name: nodejs
        state: present

    - name: Run npm install
      npm:
        path: /home/ubuntu/app/app
        name: install

    - name: Install PM2 globally using npm
      npm:
        name: pm2
        global: yes
        state: present


    - name: Stop pm2 processes
      shell: pm2 kill

    - name: Start app with pm2
      shell: pm2 start app.js
      args:
        chdir: /home/ubuntu/app/app
```
Explaination of each step:

1. **Update APT cache**: This task uses the `apt` module to update the package manager cache on the remote server.

2. **Upgrade packages**: This task uses the `apt` module again, but this time it performs a distribution upgrade, updating all installed packages to their latest available versions.

3. **Clone GitHub repository**: This task uses the `git` module to clone the specified GitHub repository to the `/home/ubuntu/app` directory on the remote server. The version "main" is used here.

4. **Install Node.js**: This task uses the `get_url` module to download the Node.js setup script from `https://deb.nodesource.com/setup_12.x` and save it as `/tmp/node_setup.sh` on the remote server.

5. **Run Node.js setup script**: This task runs the Node.js setup script using `sudo` and `-E` flag to preserve environment variables. It sets up the Node.js repository on the server.

6. **Install Node.js package**: This task uses the `apt` module to install Node.js on the remote host.

7. **Run npm install**: This task uses the `npm` module to install Node.js dependencies of the web application located in `/home/ubuntu/app/app` directory.

8. **Install PM2 globally using npm**: This task uses the `npm` module to install PM2 globally on the remote host.

9. **Stop pm2 processes**: This task stops all running PM2 processes on the remote host using the `shell` module and the command `pm2 kill`.

10. **Start app with pm2**: This task starts the application using PM2. It runs the command `pm2 start app.js` with the `chdir` option set to `/home/ubuntu/app/app`, indicating the directory where the app.js file is located.


#### Adding in reverse proxy configuration

``` 
   - name: Install sed
      apt:
        name: sed
        state: present

    - name: Configure reverse proxy in Nginx
      shell: sudo sed -i 's#try_files $uri $uri/ =404;#proxy_pass http://localhost:3000;#g' /etc/nginx/sites-available/default

    - name: Restart Nginx
      service:
        name: nginx
        state: restarted
```
Add this after the npm install but before the pm2 start.

Explanation of the configure line:

1. Name: "Configure reverse proxy in Nginx"

2. The name of the task for identification and logging purposes.
Shell command: This task executes a shell command directly on the remote host.

3. sudo: The sudo command allows the task to run with administrative privileges, necessary for modifying the Nginx configuration.

4. sed: A stream editor used for filtering and transforming text.

5. -i: The -i option performs in-place editing of the specified file.

6. Pattern and Replacement: 's#try_files $uri $uri/ =404;#proxy_pass http://localhost:3000;#g'

  - This sed command replaces the line containing try_files $uri $uri/ =404; with proxy_pass http://localhost:3000; in the Nginx configuration file.
  - It sets up a reverse proxy to forward incoming requests to a backend server running on http://localhost:3000.
  - This is often used to proxy requests to a web application server, such as Node.js, behind Nginx for additional features like SSL termination, load balancing, and caching.
7. Configuration file: /etc/nginx/sites-available/default

  - The path to the Nginx configuration file that will be modified.
  - In this case, it's the default Nginx site configuration file.

When this task is executed, it will configure Nginx to act as a reverse proxy, forwarding incoming requests to a backend server running on http://localhost:3000. This setup is useful when you have a web application (e.g., a Node.js application) running on localhost:3000, and you want to serve it through Nginx, which can handle additional features like SSL termination, load balancing, and caching.

### Creating playbook for database

- Automate provisioning and configuring
- Mongodb in db instance
- install mongodb
- ensure port 27027 is enabled in sg
- ensure mongod.conf has required configuration
- allow app and controller access
  - provide specific ips (production)
  - or bind ip = 0.0.0.0:27017 (training)
- Need to provide the enviroment variable DB_HOST=db-ip:27017/posts in the app

![Alt text](imgs/overview-steps.png)

Common blockers:

- npm not being friendly - reverse proxy issues - env not persistant - SG rule issues - mongod.conf not configured correctly 
- If there is an issue and you arent sure what is going wrong you can run the playbook command with -vvv and it will show you what is happening in the background.

Steps:

1. cd etc/ansible
2. `sudo nano mongodb-playbook.yml`
3. In here write the playbook

```
---

# create playbook to install mongodb in db-machine instance

- hosts: db
  gather_facts: yes
  become: true

  tasks:

  - name: Update
    command: sudo apt update -y

  - name: Upgrade
    command: sudo apt upgrade -y

# Install mongodb
  - name: Install Mongodb
    apt: pkg=mongodb state=present

# Ensure db is running


# check the status to see if it is running
```

`sudo ansible db -a "systemctl status mongodb"` - check if it is running

4. Add the configuration of the bind ip in the mongodb playbook

```  - name: Install sed
    apt:
      name: sed
      state: present

  - name: Configure mongodb.conf bind ip
    become: true
    shell: sudo sed -i 's/127.0.0.1/0.0.0.0/g' /etc/mongodb.conf

  - name: Restart Mongodb
    service:
      name: mongodb
      state: restarted
      enabled: yes
```

5. Add the env variable into the node playbook 

```    - name: Add DB_HOST environment variable to /etc/environment file
      lineinfile:
        path: /etc/environment
        line: 'export DB_HOST="mongodb://34.245.78.113:27017/posts"'
        state: present
        create: yes
```