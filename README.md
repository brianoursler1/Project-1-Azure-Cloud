## Automated ELK Stack Deployment

The files in this repository were used to configure the network depicted below.

![alt text](https://github.com/brianoursler1/Project-1-Azure-Cloud/blob/2c02e2dd03177684ea59a6fea1de081818c983e3/Virtual%20Machine%20Diagram%20.png)

These files have been tested and used to generate a live ELK deployment on Azure. They can be used to either recreate the entire deployment pictured above. Alternatively, select portions of the _____ file may be used to install only certain pieces of it, such as Filebeat.

ELK installation Playbook YAML file:

```
----
- name: Configure Elk VM with Docker
  hosts: elk
  remote_user: brian
  become: true
  tasks:
    # Use apt module
    - name: Install docker.io
      apt:
        update_cache: yes
        force_apt_get: yes
        name: docker.io
        state: present

      # Use apt module
    - name: Install python3-pip
      apt:
        force_apt_get: yes
        name: python3-pip
        state: present

      # Use pip module (It will default to pip3)
    - name: Install Docker module
      pip:
        name: docker
        state: present

      # Use command module
    - name: Increase virtual memory
      command: sysctl -w vm.max_map_count=262144

      # Use sysctl module
    - name: Use more memory
      sysctl:
        name: vm.max_map_count
        value: '262144'
        state: present
        reload: yes

      # Use docker_container module
    - name: download and launch a docker elk container
      docker_container:
        name: elk
        image: sebp/elk:761
        state: started
        restart_policy: always
        # Please list the ports that ELK runs on
        published_ports:
          - 5601:5601
          - 9200:9200
          - 5044:5044

      # Use systemd module
    - name: Enable service docker on boot
      systemd:
        name: docker
        enabled: yes
```

File beat Playbook YAML File

```
---
- name: installing and launching filebeat
  hosts: webservers
  become: yes
  tasks:

  - name: download filebeat deb
    command: curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-7.6.1-amd64.deb

  - name: install filebeat deb
    command: dpkg -i filebeat-7.6.1-amd64.deb

  - name: drop in filebeat.yml
    copy:
      src: /etc/ansible/files/filebeat-config.yml
      dest: /etc/filebeat/filebeat.yml

  - name: enable and configure system module
    command: filebeat modules enable system

  - name: setup filebeat
    command: filebeat setup

  - name: start filebeat service
    command: service filebeat start
```
Metric beat playbook YAML File:

```
---
- name: Install metric beat
  hosts: webservers
  become: true
  tasks:
    # Use command module
  - name: Download metricbeat
    command: curl -L -O https://artifacts.elastic.co/downloads/beats/metricbeat/metricbeat-7.6.1-amd64.deb

    # Use command module
  - name: install metricbeat
    command: dpkg -i metricbeat-7.6.1-amd64.deb

    # Use copy module
  - name: drop in metricbeat config
    copy:
      src: /etc/ansible/files/metricbeat-config.yml
      dest: /etc/metricbeat/metricbeat.yml

    # Use command module
  - name: enable and configure docker module for metric beat
    command: metricbeat modules enable docker

    # Use command module
  - name: setup metric beat
    command: metricbeat setup

    # Use command module
  - name: start metric beat
    command: metricbeat -e

```

This document contains the following details:
- Description of the Topologu
- Access Policies
- ELK Configuration
  - Beats in Use
  - Machines Being Monitored
- How to Use the Ansible Build


### Description of the Topology

The main purpose of this network is to expose a load-balanced and monitored instance of DVWA, the D*mn Vulnerable Web Application.

Load balancing ensures that the application will be highly redundant and available, in addition to restricting access to the network.
- Load balancers protect availability within the network. The advantage of the jump box is that it acts as a provisioner. Therefore, the jump box provides automated, remote configuration of the web VMs. 

Integrating an ELK server allows users to easily monitor the vulnerable VMs for changes to the virtual machine and system files.
- File beat monitors syslog events, sudo commands, SSH logins and new users and groups
- Metric beat monitors Docker container hosts, images and names for memory usage, Network IO and CPU usage.

The configuration details of each machine may be found below.

| Name                    | Function            | IP Address              | Operating System |
|-------------------------|---------------------|-------------------------|------------------|
| Jump-Box-Provisioner VM | Gateway             | 10.0.0.4/52.250.65.166  | Linux Ubuntu     |
| Web-1 VM                | DVWA Container      | 10.0.0.5/52.183.2.223   | Linux Ubuntu     |
| Web-2 VM                | DVWA Container      | 10.0.0.6/52.183.2.223   | Linux Ubuntu     |
| Web-3 VM                | DVWA Container      | 10.0.0.7/52.183.2.223   | Linux Ubuntu     |
| ELK VM                  | ELK Stack Container | 10.1.0.4/137.116.34.220 | Linux Ubuntu     |
| Load Balancer           | Load balancer       | 52.183.2.223            | NA               |
 

### Access Policies

The machines on the internal network are not exposed to the public Internet. 

Only the Jump-Box-Provisioner machine can accept connections from the Internet. Access to this machine is only allowed from the following IP addresses:
- 75.106.128.144

Machines within the network can only be accessed by the Jump-Box-Provisioner.
- This machine was accessed from my local hostmachine. IP address 75.106.128.144

A summary of the access policies in place can be found in the table below.

| Name                    | Publicly Accessible | Allowed IP Addresses                                    |
|-------------------------|---------------------|---------------------------------------------------------|
| Jump-Box-Provisioner VM | Yes                 | 10.0.0.5, 10.0.0.6, 10.0.0.7, 10.1.0.4, 75.166.128.144  |
| Web-1 VM                | No                  | 10.0.0.4                                                |
| Web-2 VM                | No                  | 10.0.0.4                                                |
| Web-3 VM                | No                  | 10.0.0.4                                                |
| ELK VM                  | No                  | 10.0.0.4                                                |

### Elk Configuration

Ansible was used to automate configuration of the ELK machine. No configuration was performed manually, which is advantageous because...
- Ansible provides an efficienct method for configuring many VMs at once. This helps a network scale quickly while providing consistancy and reducing configuration errors that would normally be found with manual configurations.

The playbook implements the following tasks:
- Uses the apt module to install docker.io
- Uses the apt module to install python3-pip
- Uses the pip module to install the Docker module
- Uses the docker_container module to download the elk image
- Uses the published_ports to list the ports ELK runs on
- Uses systemd module to enable Docker on start

The following screenshot displays the result of running `docker ps` after successfully configuring the ELK instance.

![alt text](https://github.com/brianoursler1/Project-1-Azure-Cloud/blob/b9f744f99b6345526c69e8bf284f1bd59b2532f8/Sudo%20Docker%20PS.PNG)

### Target Machines & Beats
This ELK server is configured to monitor the following machines:
- Web-1 VM: 10.0.0.5
- Web-2 VM: 10.0.0.6
- Web-3 VM: 10.0.0.7

We have installed the following Beats on these machines:
- Metric beat
- File beat

These Beats allow us to collect the following information from each machine:
- Metric Beat allows collection of CPU usage, Network IO, number of containers running and memory usage.
- File beat allows collection of SSH logins, monitoring of syslog files, sudo command usage and new users and group additions.

### Using the Playbook
In order to use the playbook, you will need to have an Ansible control node already configured. Assuming you have such a control node provisioned: 

SSH into the control node and follow the steps below:
- Copy the ELK installation Playbook file, File beat Playbook file and Metric beat playbook file contents into .yml files located in the ansible container (/etc/ansible).
- Update the ansible hosts file to assign IP addresses to host groups. In this case, a "webservers" group was created containing 10.0.0.5, 10.0.0.6 and 10.0.0.7 and an "elk" machine group containing 10.1.0.4. 
- Update the ELK installation Playbook YAML file to include the host group the ELK stack will be deployed on. This can be done by changing the "Hosts:" field to elk.
- Update the File beat and Metric beat Playbook YAML files to include the hosts to be monitored. This can be done by changing the "Hosts:" field to webservers.
- Run the ELK installation Playbook YAML file, and navigate to http://[enter public IP address of ELK VM]:5601/app/kibana to check that the installation worked as expected.

### Downloading the Playbook
1. SSH into the Jump-Box-Provisioner by running ``` ssh -i <Path to Public SSH Key> <username>@IP Address of Jump-box-provisioner ```
2. Start docker and attach the docker run ``` sudo docker start <name of docker> ``` ``` sudo docker attach <name of docker> ```
3. Navigate to /etc/ansible run ``` cd /etc/ansible ```
4. While in the /etc/ansible directory use the ``` curl ``` command to pull the desired files. Alternatively you can run ``` touch <filename> ``` to create a file then copy and paste the code in the file. However, this is not recommended. 
- I saved the install-elk.yml, ansible.cfg and host files in /etc/ansible
- I created a files folder in /etc/asnsible and saved filebeat-config.yml and metricbeat-config.yml in this folder.
- I created a roles folder in /etc/ansible and saved filebeat-playbook.yml and metricbeat-playbook.yml in this folder. 
### Editing the Playbook
1. Edit the hosts file. This can be done running ```nano hosts ``` This host file determines how ansible speaks with the virtual machines by placing them into groups. Place the virtual machines to be monitored into a single group. These machines will be installed with metric beat and file beat in the ``` [webservers] ``` group. Place the machine that will be installed with the ELK stack in the ``` [elk] ``` group. Make sure to use the private IP addresses of these machines. At the end of the IP address include ``` ansible_python_interpreter=usr/bin/python3 ```
Here is a sample of what the host file looks like after configuration

```
# This is the default ansible 'hosts' file.
#
# It should live in /etc/ansible/hosts
#
#   - Comments begin with the '#' character
#   - Blank lines are ignored
#   - Groups of hosts are delimited by [header] elements
#   - You can enter hostnames or ip addresses
#   - A hostname/ip can be a member of multiple groups

# Ex 1: Ungrouped hosts, specify before any group headers.

## green.example.com
## blue.example.com
## 192.168.100.1
## 192.168.100.10

# Ex 2: A collection of hosts belonging to the 'webservers' group

 [webservers]
## alpha.example.org
## beta.example.org
## 192.168.1.100
## 192.168.1.110
10.0.0.5 ansible_python_interpreter=/usr/bin/python3
10.0.0.6 ansible_python_interpreter=/usr/bin/python3
10.0.0.7 ansible_python_interpreter=/usr/bin/python3

[elk]
10.1.0.4 ansible_python_interpreter=/usr/bin/python3
# If you have multiple hosts following a pattern you can specify
# them like this:

## www[001:006].example.com

# Ex 3: A collection of database servers in the 'dbservers' group

## [dbservers]
##
## db01.intranet.mydomain.net
## db02.intranet.mydomain.net
```
2. Edit the ansible.cfg file. This can be done by running ``` nano ansible.cfg ``` Navigate to line 107 and designate a remote user name. For simplicity I used the same user name created when the VMs were setup in Azure. I also used the same user name for all VMs to keep things easy to manage.
3. Edit the install-elk.yml file. This can be done by running ``` nano install-elk.yml ```. 
### Running the Playbook
### Confirming Playbook Success
_As a **Bonus**, provide the specific commands the user will need to run to download the playbook, update the files, etc._
