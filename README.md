# project1
Project 1 submission
## Automated ELK Stack Deployment

The files in this repository were used to configure the network depicted below.

![diagram](https://github.com/srirajb3/project1/blob/main/diagrams/Elk_Diagram.png)

These files have been tested and used to generate a live ELK deployment on Azure. They can be used to either recreate the entire deployment pictured above. Alternatively, select portions of the playbook file may be used to install only certain pieces of it, such as Filebeat.

Playbook 1 : playbook.yml
 ---
  - name: config web vm with docker
    hosts: webservers
    become: true
    tasks:
    - name: docker.io
      apt:
        force_apt_get: yes
        update_cache: yes
        name: docker.io
        state: present

    - name: install pip3
      apt:
        force_apt_get: yes
        name: python3-pip
        state: present

    - name: Install docker python module
      pip:
        name: docker
        state: present

    - name: download and launch a docker web container
      docker_container:
        name: dvwa
        image: cyberxsecurity/dvwa
        state: started
        published_ports: 80:80

    - name: enable docker service
      systemd:
        name: docker
        enabled: yes

Playbook 2: install-elk.yml
 ---
  - name: config elk with docker0
    hosts: elk
    become: true
    tasks:
    - name: Config_map_count
      command: sysctl -w vm.max_map_count=262144

    - name: docker.io
      apt:
        force_apt_get: yes
        update_cache: yes
        name: docker.io
        state: present

    - name: install pip3
      apt:
        force_apt_get: yes
        name: python3-pip
        state: present

    - name: install docker python module
      pip:
        name: docker
        state: present

    - name: download and launch a docker web container
      docker_container:
        name: elk
        image: sebp/elk:761
        state: started
        restart_policy: always
        published_ports:
          - 5601:5601
          - 9200:9200
          - 5044:5044

    - name: enable docker service
      systemd:
        name: docker
        enabled: yes

Playbook 3 : filebeat-playbook.yml
 ---
  - name: installing and launching filebeat
    hosts: webservers
    become: true
    tasks:

    - name: download filebeat deb
      command: curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-7.6.1-amd64.deb

    - name: install filebeat deb
      command: sudo dpkg -i filebeat-7.6.1-amd64.deb

    - name: drop in filebeat.yml
      copy:
        src: /etc/ansible/filebeat-config.yml
        dest: /etc/filebeat/filebeat.yml

    - name: enable and configure system module
      command: sudo filebeat modules enable system

    - name: setup filebeat
      command: sudo filebeat setup

    - name: start filebeat service
      command: sudo service filebeat start

    - name: enable service filebeat on boot
      systemd:
        name: filebeat
        enabled: yes

This document contains the following details:
- Description of the Topologu
- Access Policies
- ELK Configuration
  - Beats in Use
  - Machines Being Monitored
- How to Use the Ansible Build


### Description of the Topology

The main purpose of this network is to expose a load-balanced and monitored instance of DVWA, the D*mn Vulnerable Web Application.

Load balancing ensures that the application will be highly accesible, in addition to restricting access to the network.
- Load balancers can help protect websites, user access and protect agaisnt DDoS attacks. They also protect application availability. 
- A jump box requires all connections to a cloud network to come through a single VM. This allows for easier monitoring of remote connections.

Integrating an ELK server allows users to easily monitor the vulnerable VMs for changes to the configuration and system files.
- Filebeat is utilized to monitor specified log files and collects log events.
- Metricbeat montiors systems and services.

The configuration details of each machine may be found below.
_Note: Use the [Markdown Table Generator](http://www.tablesgenerator.com/markdown_tables) to add/remove values from the table_.

| Name       | Function | IP Address | Operating System |
|------------|----------|------------|------------------|
| Jump Box   | Gateway  | 10.0.0.1   | Linux            |
| Web-1      | DVWA     | 10.0.0.8   | Linux            |
| Web-2      | DVWA     | 10.0.0.7   | Linux            |
| Elkmachine | ELK      | 10.1.0.4   | Linux            |

### Access Policies

The machines on the internal network are not exposed to the public Internet. 

Only the Jump Box machine can accept connections from the Internet. Access to this machine is only allowed from the following IP addresses:
- 13.68.157.78

Machines within the network can only be accessed by Jump Box.
- The Jump Box can access the Elk VM through IP 10.1.0.4

A summary of the access policies in place can be found in the table below.

| Name     | Publicly Accessible | Allowed IP Addresses |
|----------|---------------------|----------------------|
| Jump Box | Yes - SSH           | 73.176.253.251       |
| Web-1    | Yes - HTTP          | 73.176.253.251       |
| Web-2    | Yes - HTTP          | 73.176.253.251       |
| Elk      | Yes - HTTP          | 73.176.253.251       |

### Elk Configuration

Ansible was used to automate configuration of the ELK machine. No configuration was performed manually, which is advantageous because.
- Automation with ansible simplifies complex tasks, streamlines OS and software update
- Sitewide security policies can be configured and deployed rapidly.

The playbook implements the following tasks:

Playbook 1: playbook.yml

playbook.yml is used to setup the DVWA webservers.
- Installs docker
- Installs python
- Installs docker python module
- Downloads and launches a docker web container
- Enables docker service

Playbook 2: install-elk.yml

install-elk.yml is used to setup the ELK repository.
- Installs docker
- Installs python
- Installs dockery python module
- Downloads and launches a docker web container
- Enables docker service

The following screenshot displays the result of running `docker ps` after successfully configuring the ELK instance.

https://github.com/srirajb3/project1/blob/main/diagrams/Elk%20Docker%20PS%20image.PNG

### Target Machines & Beats
This ELK server is configured to monitor the following machines:
- Web-1 10.0.0.8 
- Web-2 10.0.0.7

We have installed the following Beats on these machines:
- Filebeat
- Metricbeat

These Beats allow us to collect the following information from each machine:
- Filebeats collects specified logs from VM's running filebeat 
- Metricbeat collects metrics from the operating system and services from VM's running metricbeat


### Using the Playbook
In order to use the playbook, you will need to have an Ansible control node already configured. Assuming you have such a control node provisioned: 

SSH into the control node and follow the steps below:
- Copy the playbook file to ansible docker container.
- Update the ansible.cfg file to include remote_user parameter to srimachine1
- Run the playbook, and navigate to Kibana to check that the installation worked as expected.



_As a **Bonus**, provide the specific commands the user will need to run to download the playbook, update the files, etc._
