[TOC]
- [General](#general)
- [Requirements](#requirements)
- [Vagrant](#vagrant)
- [Ansible](#ansible)
- [Kubernetes](#kubernetes)
- [TODO](#todo)


# General

This code was written so we can test kubernetes clusters on local machines. But the main purpose of this code is to create kubernetes clusters on bare metal machines. Our goal was to only have master nodes in the cluster and that's why you won't see code for worker nodes. Adding provisioning of worker nodes to this playbook should be easy as most of the code is independent.

# Requirements

What you need to have installed on ansible host:
* ansible
* kubectl
* helm

# Vagrant

- See [Vagrant.md](Vagrant.md)

# Ansible

If you don't want to run vagrant you can just go to ansible and  modify the `hosts.example` file

**This playbook only works on CentOS/RedHat**

```sh
cd ansible
cp hosts.example hosts
# Modify the hosts file to your needs
ansible-playbook -i hosts playbook.yml
```

# Kubernetes

Here is how it works:
> keepalived -> haproxy -> kube-apiserver

In this playbook we are only creating etcd cluster and k8s master nodes in HA. We are not creating worker nodes. Worker nodes will be added later the the playbook.

Here are the following steps:
1. Installing all dependencies
2. Installing **docker-ce**
3. Installing **haproxy**
4. Installing **keepalived**
5. Installing **etcd** and joining all the nodes together
6. Installing kube* and creating first master
   1. Joining other masters to first master
7. Copying kube config to local machine from where the ansible was executed
8. Printing message


# TODO

1. Masters role have hardcoded `creates` checks to `vagrant` directory... this should be changed!

> Example:
>```
>  args:
>    creates: /home/vagrant/.kube/config
>```

2. Add worker nodes
