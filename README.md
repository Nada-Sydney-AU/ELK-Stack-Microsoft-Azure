# Automated ELK Stack Deployment

This document contains the following details:
- Description of the Red-Team Network Topology
- ELK Configuration
  - Beats in Use
  - Machines Being Monitored
- How to Use the Ansible Build
- Access Policies

### Description of the Topology
This repository includes code defining the infrastructure below. 

![Red-Team AZURE](https://github.com/Nada-Sydney-AU/ELK-Stack-Microsoft-Azure/blob/master/Network%20Diagrams/Red-Team_AZURE_NRM.png)

The main purpose of this network is to expose a load-balanced and monitored instance of DVWA, the "D*mn Vulnerable Web Application"

Load balancing ensures that the application will be highly available, in addition to restricting inbound access to the network. 

Load balancers serve to distribute network traffic and application workloads across the Web servers so that not one is overwhelmed.

Access controls ensure that only authorised users will be able connect via secure connection from the Host Machine IP to the Jump Box (Gateway VM).

Integrating an ELK server allows users to easily monitor the vulnerable VMs for changes to the file systems of the VMs on the network, as well as watch system metrics, such as CPU usage; attempted SSH logins; `sudo` escalation failures.

The configuration details of each machine may be found below.

| Name     |   Function  | IP Address |Public IPv4   | Operating System |
|----------|-------------|------------|--------------|------------------|
| Jump Box | Gateway     | 10.0.0.4   |20.188.210.238| Linux            |
| DVWA 1   | Web Server  | 10.0.0.5   |              | Linux            |
| DVWA 2   | Web Server  | 10.0.0.6   |              | Linux            |
| DVWA 3   | Web Server  | 10.0.0.7   |              | Linux            |
| ELK      | Monitoring  | 10.1.0.4   |              | Linux            |

### Access Policies
The machines on the internal network are _not_ exposed to the public Internet. 

Only the Jump Box machine can accept connections from the Internet. Access to this machine is only allowed from the IP address `27.33.14.54`


Machines _within_ the network can only be accessed by each other. The DVWA VMs send traffic to the ELK server.

A summary of the access policies in place can be found in the table below.

| Name     | Publicly Accessible | Allowed IP Addresses |
|----------|---------------------|----------------------|
| Jump Box | Yes                 | 27.33.14.54          |
| ELK      | No                  | 10.1.0.1-254         |
| DVWA 1   | No                  | 10.0.0.1-254         |
| DVWA 2   | No                  | 10.0.0.1-254         |
| DVWA 3   | No                  | 10.0.0.1-254         |

## ELK Server Configuration
The ELK VM exposes an Elastic Stack instance. **Docker** is used to download and manage an ELK container.

To set up the ELK VM we developed a reusable Ansible Playbook to automate the configuration proccess. The Ansible Playbook we create can be resused allowing the configuration of additional VM's to be created by simply running the Playbook. 

Ansible was used to automate configuration of the ELK machine. No configuration was performed manually, which is advantageous because...
- _TODO: What is the main advantage of automating configuration with Ansible?_

The playbook implements the following tasks:
This playbook automates the following tasks:

 - Install docker
 - Install apt module
 - Install pip3
 - Install pip module
 - Download and launch a docker container


Login into to Jump-box run `ansible-playbook install_elk.yml elk`
This command runs the `install_elk.yml` playbook on the `elk` host.

The following screenshot displays the result of running docker ps after successfully configuring the ELK instance. (Images/DockerPs.png)

```bash
$ sudo docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                  PORTS                                                                              NAMES
217ed9a25488        sebp/elk:761        "/usr/local/bin/star…"   6 days ago          Up Less than a second   0.0.0.0:5044->5044/tcp, 0.0.0.0:5601->5601/tcp, 0.0.0.0:9200->9200/tcp, 9300/tcp   elk
```

!Docker Container](Images/docker_ps.png)

The playbook is duplicated below.

```yaml
---
# install_elk.yml
- name: Configure Elk VM with Docker
  hosts: elkservers
  remote_user: elk
  become: true
  tasks:
    # Use apt module
    - name: Install docker.io
      apt:
        update_cache: yes
        name: docker.io
        state: present

      # Use apt module
    - name: Install pip3
      apt:
        force_apt_get: yes
        name: python3-pip
        state: present

      # Use pip module
    - name: Install Docker python module
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
        value: "262144"
        state: present
        reload: yes

      # Use docker_container module
    - name: download and launch a docker elk container
      docker_container:
        name: elk
        image: sebp/elk:761
        state: started
        restart_policy: always
        published_ports:
          - 5601:5601
          - 9200:9200
          - 5044:5044
```

### Target Machines & Beats
This ELK server is configured to monitor the following machines:

-DVWA 1 at 10.0.0.5
-DVWA 2 at 10.0.0.6
-DVWA 3 at 10.0.0.7
We have installed the following Beats on these machines:
- Filebeat
- Metricbeat
- Packetbeat

These Beats allow us to collect the following information from each machine:
- **Filebeat**: Filebeat detects changes to the filesystem. Specifically, we use it to collect Apache logs.
- **Metricbeat**: Metricbeat detects changes in system metrics, such as CPU usage. We use it to detect SSH login attempts, failed `sudo` escalations, and CPU/RAM statistics.
- **Packetbeat**: Packetbeat collects packets that pass through the NIC, similar to Wireshark. We use it to generate a trace of all activity that takes place on the network, in case later forensic analysis should be warranted.

The playbook below installs Metricbeat on the target hosts. The playbook for installing Filebeat is not included, but looks essentially identical — simply replace `metricbeat` with `filebeat`, and it will work as expected.

```yaml
---
- name: Install metric beat
  hosts: webservers
  become: true
  tasks:
    # Use command module
  - name: Download metricbeat
    command: curl -L -O https://artifacts.elastic.co/downloads/beats/metricbeat/metricbeat-7.4.0-amd64.deb

    # Use command module
  - name: install metricbeat
    command: dpkg -i metricbeat-7.4.0-amd64.deb

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
    command: service metricbeat start
```

### Using the Playbooks
In order to use the playbooks, you will need to have an Ansible control node already configured. We use the **Jump Box** for this purpose.

To use the playbooks, we must perform the following steps:
- Copy the playbooks to the Ansible Control Node
- Run each playbook on the appropriate targets

The easiest way to copy the playbooks is to use Git:

```bash
$ cd /etc/ansible
$ mkdir files
# Clone Repository + IaC Files
$ git clone https://github.com/yourusername/project-1.git
# Move Playbooks and hosts file Into `/etc/ansible`
$ cp project-1/playbooks/* .
$ cp project-1/files/* ./files
```

This copies the playbook files to the correct place.

Next, you must create a `hosts` file to specify which VMs to run each playbook on. Run the commands below:

```bash
$ cd /etc/ansible
$ cat > hosts <<EOF
[webservers]
10.0.0.5
10.0.0.6
10.0.0.7

[elk]
10.1.0.4
EOF
```

After this, the commands below run the playbook:

 ```bash
 $ cd /etc/ansible
 $ ansible-playbook install_elk.yml elk
 $ ansible-playbook install_filebeat.yml webservers
 $ ansible-playbook install_metricbeat.yml webservers
 ```

To verify success, wait five minutes to give ELK time to start up. 

Then, run: `curl http://10.1.0.4:5601`. This is the address of Kibana. If the installation succeeded, this command should print HTML to the console.

_TODO: Answer the following questions to fill in the blanks:_
- _Which file is the playbook? Where do you copy it?_
- _Which file do you update to make Ansible run the playbook on a specific machine? How do I specify which machine to install the ELK server on versus which to install Filebeat on?_
- _Which URL do you navigate to in order to check that the ELK server is running?

_As a **Bonus**, provide the specific commands the user will need to run to download the playbook, update the files, etc._
