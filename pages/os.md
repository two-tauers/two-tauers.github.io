---
layout: page
title: OS
permalink: /os/
nav_order: 2
---

## Install Ubuntu Server

This is the least automated part.

1. Download the [Raspberry Pi Imager](https://www.raspberrypi.com/software/)

2. Insert an SD card into a PC/laptop and run the imager.

3. Choose the SD card as the storage and Ubuntu Server 20.04 (64-bit server OS for arm64 architectures). The reason this OS is better than the Raspbian, is that it is arm64 and therefore has better support in the community. You can, of course, choose any OS, but some might have peculiarities that won't be covered by these docs or ansible playbooks.

4. Click Write and wait. It might take between 10 and 30 min depending on your hardware.

5. Repeat for each of the SD cards

## Configure the OS

Before we install the kubernetes cluster, the OS will need some configurations.
The most obvious one is the fact that Ubuntu requires default user password change on first login.
We don't want that user anyway as we will be creating our own users, but it is enforced.
There are a few more useful tweaks like disabling bluetooth and wifi to not waste energy, giving appropriate hostnames, setting a timezone etc.

Configuration scripts live in the [baseline repo](https://github.com/two-tauers/baseline/).
The readme briefly explains what the code is doing, but the two main sections are Prerequisites and Usage.

### Prerequisites

1. [Install ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html). This is the main tool for configuration of the OS and installation of k3s.

2. [Install sshpass](https://www.cyberciti.biz/faq/noninteractive-shell-script-ssh-password-provider/). As Ubuntu enforces password change on first login, we will need a way for ansible to access it with the default password and changing it before we create users with passwordless access.

3. Static IPs for the nodes. The process will differ for different routers, so if you get stuck here, please refer to your router's documentation. By default, all devices will get an IP address assigned by your router when connected to the network. For the cluster to be able to talk to each other without finding out their IP addresses manually, we will need to give them static IP addresses. To do that, you will need to find out MAC addresses of the Raspberry Pis:

    - Connect a single Raspberry Pi to the network
    - Log in to your router's management app (usually at http://192.168.0.1/, but it might differ for your model, RTFM)
    - Go to the list of connected devices and note the MAC address of the one called 'ubuntu'
    - Go to the DHCP section and reserve an IP address for this device
    - Repeat for the other Pis

    You can choose any IP addresses in the allowed range, but it is helpful to make them memorable.
    I went for 192.168.0.100 for the master node and 192.168.0.101-103 for the worker nodes.

### Run the baseline playbooks

Now that you have IP addresses of all the nodes, we can set up ansible inventory.
Below is the example from the repo and some explanations

```yaml
---
# group of ALL hosts
all:

  # variables that apply to all hosts
  vars:
  
    # user that ansible will use to manage the hosts
    ansible_ssh_user: ansible

    # path to an ssh key that ansible user will use to access the hosts
    ansible_ssh_private_key_file: 
      {%raw%}"{{ lookup('env','HOME') }}/.ssh/two-tauers-ansible"{%endraw%}

    # timezone of the hosts
    timezone: "Europe/London"

    # whether or not to change default user's password
    # if yes, ansible will log in with the defaul ubuntu credentials (user:ubuntu, passsword:ubuntu) and change the password to 'dummy-password'
    # this is only changed to something in order to have access to create the users and the default user is disabled afterwards
    # if no, it will try accessing the hosts with 'dummy-password'
    perform_initial_password_change: yes
    
    # create users as listed below if they don't exist
    create_users: yes 

    # users to create
    # ansible for ansible access using the key path mentioned above
    # tt (rename this to your liking) for your personal access
    # Note that if the keys don't exist, they will be created locally
    users:
      - username: ansible
        sudoer: yes
        key_path: {%raw%}"{{ ansible_ssh_private_key_file }}"{%endraw%}
      - username: tt
        sudoer: yes
        key_path: {%raw%}"{{ lookup('env', 'HOME') }}/.ssh/id_ed25519"{%endraw%}

  # list of hosts - names and IP addresses
  # change names to your liking and IP addresses to the IP addresses of the hosts that your set in the above step
  hosts:
    sauron:
      ansible_host: 192.168.0.100
    orker1:
      ansible_host: 192.168.0.101
    orker2:
      ansible_host: 192.168.0.102
    orker3:
      ansible_host: 192.168.0.103
```

After the inventory is defined, run

```bash
ansible-playbook site.yaml
```

and wait for it. At the end of the playbook it will restart all the hosts to apply the changes.

Now you're ready to install k3s!
