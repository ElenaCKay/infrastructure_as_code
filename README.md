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
17. ansible ping command `sudo ansible all -m ping` - doesnt work yet as host file is empty

![Alt text](imgs/ping-command.png)

18. `sudo nano hosts`

![Alt text](imgs/add-ip-host.png)

Addition in to the hosts file:

    [web]
    ec2-instance ansible_host=54.155.38.93 ansible_user=ubuntu ansible_ssh_private_key_file=~/.ssh/tech241.pem

The ansible_host is the public app ip address.

19.  Run `sudo ansible web -m ping`
    
![Alt text](imgs/ping-working.png)


controller ip: 52.30.95.146
app ip: 54.155.38.93
db ip: 54.195.61.33



## Orchestration with Terraform

Not using AWS cloud formation as this could then only be used on AWS. It is cloud dependent. Terraform is cloud independant and so it can be used on any cloud, local or hybrid. 