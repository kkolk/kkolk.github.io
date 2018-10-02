---
layout: post
title:  "Installing Microsoft SQL Server using Ansible"
categories: ansible windows
---

Depending on how your development and test environments are managed, installing Microsoft SQL Server may be a common task, or at least a complex task for a Windows Administrator.   Ansible can provide a way to make this task consistant and repeatable.  Using the excellent package of DBATools (https://dbatools.io/) we can even use Ansible to execute complex configuration steps.  I have posted the role developed in this guide to Ansible Galaxy for ease of reuse.  However, this guide will cover indepth the individual steps and how they operate.
<!--more-->

## Getting Started

This guide builds on the previous environment outlined in [Configure an Ansible testing system on Windows](https://www.frostbyte.us/configure-an-ansible-testing-system-on-windows-part-1/).  For the purposes of this guide we will adjust the member server role in that enviroment to be our SQL Server.  You will also need to download a copy of [Microsoft SQL Server Developer Edition 2017](https://go.microsoft.com/fwlink/?linkid=853016).

We also need to adjust our Vagrant file to add some memory to our future SQL Server, the update Vagrant file is shown below:

```Ruby
Vagrant.configure("2") do |config|
  config.vm.define "dc" do |dc|
    dc.vm.box = "kkolk/w2k12r2-sysprep-ready"
    dc.vm.network "private_network", ip: "192.168.100.10"
  end
  config.vm.define "server" do |server|
    server.vm.provider "virtualbox" do |vb|
      vb.memory = "4096"
      vb.cpus = "1"
    end
    server.vm.box = "kkolk/w2k12r2-sysprep-ready"
    server.vm.network "private_network", ip: "192.168.100.11"
  end
  config.vm.provision "shell", path:"c:/source/ansible-for-windows/setup/scripts/ConfigureRemotingForAnsible.ps1"
end
```

At this point if the environment is not provisioned at all, you can go ahead and do:

```Bash
vagrant up
```

Otherwise, the next best solution is to simply destroy the server node and recreate it:

```Shell
vagrant destroy server
vagrant up
```

At this point we can quickly configure the member server with our existing Ansible playbook:

```Bash
ansible-playbook member_server.yml -i environments/test/hosts
```

Now we are ready to get on with building our SQL Role.  Create a **mssql** folder in the **roles** folder, then create a **tasks** folder inside it, with a **main.yml** file and open it for editing.

## Powershell DSC Modules

First, we're going to make sure that our server has all the required Powershell DSC modules:

```YAML
# Load required powershell modules
- name: Powershell | Check for required Powershell DSC modules
  win_psmodule:
    name: {{ item }}
    state: present
  with_items:
    - SQLServerDsc
    - StorageDsc
    - ServerManager
    - dbatools
    - xNetworking
```
Then

