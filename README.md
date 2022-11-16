# Buid a three nodes LSF HPC cluster in a local Windows 10 desktop

Create a three nodes LSF HPC cluster on local windows desktop by using [**vagrant**](https://www.vagrantup.com/) and [**ansible**](https://www.ansible.com/).
- It can be used for POC or Sandbox test environment.
- it's a pure *100%* code based (IaC). 

## Prerequisite:
- Reference this Repository to setup [local vagrant VM development environment](https://github.com/yjun-001/vagrant_vm_windows10)
- Preload  binary  [the IBM Spectrum LSF Suite Community Edition](https://www-01.ibm.com/marketing/iwm/iwm/web/dispatcher.do?source=swerpzsw-lsf-3&mhsrc=ibmsearch_a&mhq=lsf%20install), and unzip to ansible/downloads folder

## Cluster Inventory file:
https://github.com/yjun-001/vm_cluster_lsf/blob/9d64ed70b34aebb6e26b24a9bcd84e1c448d2a50/ansible/inventory/hosts#L1-L14

### LSF Cluster Diagram:
<img src="https://github.com/yjun-001/vm_cluster_lsf/blob/55bc8396dcac5fffc17dadd89bdf3420863478d4/images/vagrant-LSF-cluster-2022-11-11-1510.svg">

## Cluster Nodes build/provision process:
- vagrant create three virtual box VMs (one master node, two nodes)
  - using vagrant ubuntu 20.04 image 
- Once VMs generated, ansible playbook kick-in for post-configuration
  - add cluster admin user (lsfadmin)
  - update /etc/hosts by hostname and cluster IPs on each nodes
  - exchange ssh public keys between servers and enable SSH Authentication between each hosts, so each host can ssh other without password
  - update each know_hosts file, so ssh login without anonnying prompt
- Install IBM Spectrum LSF package on each nodes
  - create LSF_TOP directory
  - untar lsf10.1_lsfinstall_linux_x86_64.tar.Z
  - config install.config
  - run lsfinstall -f install.config
  - run hostsetup --top="/opt/lsf" --boot="y" --setuid
  - config /etc/lsf.sudoers
  - update profile
  - restart lsfd deamon
  - verify the lsf service running
- [] Todo: 
  - copy test job find_prime.sh (preload in /vagrant/downloads/)
  - submit it to cluster, and get the results
   

## In action: Create/Run/destroy the cluster
- to create and run the cluste
```bash
...
lsfadmin@master:~$ lsadmin ckconfig -v

Checking configuration files ...


EGO 3.4.0 build 600545, Jun 10 2021
Copyright IBM Corp. 1992, 2016. All rights reserved.
US Government Users Restricted Rights - Use, duplication or disclosure restricted by GSA ADP Schedule Contract with IBM Corp.

  binary type: linux2.6-glibc2.3-x86_64
  notes: IBM Spectrum LSF Community Edition
Reading configuration from /opt/lsf/conf/lsf.conf
Nov 16 19:06:08 2022 39300 6 3.4.0 Lim starting...
Nov 16 19:06:08 2022 39300 6 3.4.0 LIM is running in advanced workload execution mode.
Nov 16 19:06:08 2022 39300 6 3.4.0 Master LIM is not running in EGO_DISABLE_UNRESOLVABLE_HOST mode.
Nov 16 19:06:08 2022 39300 5 3.4.0 /opt/lsf/10.1/linux2.6-glibc2.3-x86_64/etc/lim -C
Nov 16 19:06:08 2022 39300 6 3.4.0 LIM is running as IBM Spectrum LSF Community Edition.
Nov 16 19:06:08 2022 39300 6 3.4.0 reCheckClass: numhosts 3 so reset exchIntvl to 15.00
Nov 16 19:06:08 2022 39300 6 3.4.0 Checking Done.
---------------------------------------------------------
No errors found.

lsfadmin@master:~$ lsload
HOST_NAME       status  r15s   r1m  r15m   ut    pg  ls    it   tmp   swp   mem
master              ok   0.0   0.7   0.4  10%   0.1   2     0  112G  1.9G  1.4G
node1               ok   0.0   0.2   0.2   9%   0.0   0     1  112G  1.9G  1.6G
node2               ok   0.0   0.4   0.1   9%   0.0   0     1  112G  1.9G  1.6G
```
- to destory the cluster
```bash
>vagrant destroy
    master: Are you sure you want to destroy the 'master' VM? [y/N] y
==> master: Forcing shutdown of VM...
==> master: Destroying VM and associated drives...
    node2: Are you sure you want to destroy the 'node2' VM? [y/N] y
==> node2: Forcing shutdown of VM...
==> node2: Destroying VM and associated drives...
    node1: Are you sure you want to destroy the 'node1' VM? [y/N] y
==> node1: Forcing shutdown of VM...
==> node1: Destroying VM and associated drives...
```
