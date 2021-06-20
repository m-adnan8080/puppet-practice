========================
Foreman Ansible Playbook
========================

Ansible playbook to deploy a complete up and running Foreman and puppet-master instance within
minutes.

Features
========
The goal of this playbook is to offer a fully automated way to deploy a
complete and ready-to-use Foreman instance within minutes.

It contains multiple different roles with numerous customizable variables,
which provide the following features:

* setup webserver (apache as a proxy with ssl)
* setup isc-dhcp-server
* setup TFTP server
* setup foreman-proxy
* setup Foreman including configuration (templates, hosts, domains, etc.)

Please note that at the current time the following distributions are supported:

* Debian 7 & 8
* Ubuntu 14.04 & 16.04
* CentOS 6 & 7
* Red Hat Enterprise Linux 6 & 7

Requirements
============
The target machine should fulfill the following requirements before the
playbook is applied:

* FQDN configured
* SELinux disabled
* Required ports 67, 69, 80, 443, etc. open
* Internet and repository access (e.g. Red Hat Optional repository)

**Ansible 2.0+ is required to use this playbook!**

Installation
============
Below the required steps to execute the default playbook:

1. Clone this repository
2. Initialize the submodules containing the foreman-yml repository: ::

   $ git submodule update --init

3. Install and configure Ansible to manage the target server
4. Create an inventory file containing either the hostname or IP address of
   target machine: ::

    $ echo "$TARGET_IP" > /tmp/inventory

5. Use the playbook foreman.yml to deploy a default setup TFTP, DHCP and foreman-proxy: ::

    $ ansible-playbook foreman.yml -i /tmp/inventory -u root

6. After a successful deployment you should be able to access Foreman through http://$TARGET_IP/.

The password of the ``admin`` user is by default set to ``foreman``. The password can be updated using defaults/main.yml file by setting foreman_admin_password = SomeStrongP@ssWD

Roles
=====
Below a short overview of all included roles:

+-----------------+----------------------------------------------------+
| Name            | Description                                        |
+=================+====================================================+
| common          | update apt cache                                   |
+-----------------+----------------------------------------------------+
| foreman         | add repos and install Foreman                      |
+-----------------+----------------------------------------------------+
| foreman_proxy   | add repos, install and configure foreman-proxy     |
+-----------------+----------------------------------------------------+
| foreman_yml     | configure the Foreman instance with `foreman-yml`_ |
+-----------------+----------------------------------------------------+
| isc_dhcp_server | install and configure isc-dhcp-server              |
+-----------------+----------------------------------------------------+
| Apache          | apache reverse proxy + self signed SSL certificate |
+-----------------+----------------------------------------------------+
| tftp            | install and setup TFTP including PXE boot files    |
+-----------------+----------------------------------------------------+

Foreman Libvirt provisioner
===========================
1. Login as foreman user on foreman server::

    $ sudo su - foreman -s /bin/bash

2. Create ssh-keys for foreman user::

    $ ssh-keygen

3. Copy foreman user ssh public key to libvirt server root user and exit foreman user::

    $ ssh-copy-id root@<libvirt_server_ip>

4. Install libvirt-client on foreman server::

    $ sudo apt install -y foreman-libvirt libvirt-client

5. Testing the connection with libvirt server::

    $ sudo su foreman -s /bin/bash -c 'virsh -c qemu+ssh://root@<libvirt_server_ip>/system list'

6. Configure the foreman to use libvirt provisioner using foreman Web UI.::
      Infrastructure -> Compute Resources -> Create Compute Resource

      Name: Provide_Provisioner_Name
      Provider: Libvirt
      Description: Description
      Url: qemu+ssh://root@<libvirt_server_ip>/system

      Leave rest as default and save the settings

License
=======
GNU GENERAL PUBLIC LICENSE Version 3