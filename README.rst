Automated Installer
===================

Foreman/Puppet Master Ansible Playbook
======================================

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

6. After a successful deployment you should be able to access Foreman
through `http://$TARGET_IP/`.

The password of the ``admin`` user is by default set to ``foreman``.
The password can be updated using defaults/main.yml file by setting
``foreman_admin_password = SomeStrongP@ssWD``

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
| apache          | aapache reverse proxy and SSL/TLS certificate      |
+-----------------+----------------------------------------------------+
| tftp            | install and setup TFTP including PXE boot files    |
+-----------------+----------------------------------------------------+


Foreman Libvirt provisioner
===========================
1. Login as foreman user on foreman server ::

    $ sudo su - foreman -s /bin/bash

2. Create ssh-keys for foreman user ::

    $ ssh-keygen

3. Copy foreman user ssh public key to libvirt server root user and exit foreman user ::

    $ ssh-copy-id root@<libvirt_server_ip>

4. Install libvirt-client on foreman server ::

    $ sudo apt install -y foreman-libvirt libvirt-client

5. Testing the connection with libvirt server ::

    $ sudo su foreman -s /bin/bash -c 'virsh -c qemu+ssh://root@<libvirt_server_ip>/system list'

6. Configure the foreman to use libvirt provisioner using foreman Web UI. ::

      Infrastructure -> Compute Resources -> Create Compute Resource

      Name: Provide_Provisioner_Name
      Provider: Libvirt
      Description: Description
      Url: qemu+ssh://root@<libvirt_server_ip>/system

      Leave rest as default and save the settings

License
=======
GNU GENERAL PUBLIC LICENSE Version 3

# Manual Installation process
## Install Puppet Server on Ubuntu

sudo nano /etc/hosts

```
[puppet master ip] puppetmaster puppet
[puppet client ip] puppetclient
```

wget https://apt.puppetlabs.com/puppet6-release-$(lsb_release -cs).deb

sudo dpkg -i puppet6-release-$(lsb_release -cs).deb

sudo apt-get update -y

sudo apt-get install puppetserver pdk -y

sudo nano /etc/default/puppetserver

```
JAVA_ARGS="-Xms1g -Xmx1g -Djruby.logger.class=com.puppetlabs.jruby_utils.jruby.Slf4jLogger"
```

sudo systemctl enable --now puppetserver

sudo /opt/puppetlabs/bin/puppetserver ca list --all
sudo /opt/puppetlabs/bin/puppetserver ca sign --all

sudo /opt/puppetlabs/bin/puppetserver ca sign --certname <Agent Name>
sudo /opt/puppetlabs/bin/puppetserver ca clean --certname <Agent Name>

### Install foreman on puppetmaster server to manage agents on GUI.

echo "deb http://deb.theforeman.org/ focal 2.5" | sudo tee /etc/apt/sources.list.d/foreman.list
echo "deb http://deb.theforeman.org/ plugins 2.5" | sudo tee -a /etc/apt/sources.list.d/foreman.list
sudo apt-get -y install ca-certificates
wget -q https://deb.theforeman.org/pubkey.gpg -O- | sudo apt-key add -

sudo apt-get update && sudo apt-get -y install foreman-installer

sudo foreman-installer

## Install Puppet Agent on Debian / RedHat based OS
### Install Puppet Agent Ubuntu

wget https://apt.puppetlabs.com/puppet6-release-focal.deb

sudo dpkg -i puppet6-release-focal.deb

sudo apt-get update -y

sudo apt-get install puppet-agent -y

sudo nano /etc/puppetlabs/puppet/puppet.conf

```
[main]
certname = puppetclient
server = puppetmaster
```

sudo systemctl enable --now puppetserver

sudo /opt/puppetlabs/bin/puppet agent --test

### Install Puppet Agent CentOS

sudo rpm -Uvh https://yum.puppet.com/puppet6/puppet6-release-el-7.noarch.rpm

sudo yum install -y puppet-agent

sudo nano /etc/puppetlabs/puppet/puppet.conf

```
[main]
certname = puppetclient
server = puppetmaster
```

sudo systemctl enable --now puppetserver

sudo /opt/puppetlabs/bin/puppet agent --test

=========================================================================================
## Puppet server module creation process

### Step1
cd /etc/puppetlabs/code/environment/production/modules

pdk new module nginx

cd nignx

pdk new class install

`nano modules/nginx/manifests/install.pp`

```
# @summary Install Nginx
#
# This class install nignx web server on Debian and RedHat bases OS
#
# @example
#   include nginx::install
class nginx::install {
  package { 'install_nginx':
    name   => 'nginx',
    ensure => 'present',
  }
}
```

### To validate the class file.
puppet parser validate manifests/install.pp

### To create init.pp manifest
pdk new class nginx

`nano manifests/init.pp`

```
# @summary Install and configures Nginx
#
# Install, configures and setup a virtual hosts for nginx web server
#
# @example
#   include nginx
class nginx {
  contain nginx::install
}
```

cd /etc/puppetlabs/code/environment/production/manifests

`nano site.pp`

```
node puppetagent1.test {
  class { 'nginx': }
}
```




=========================================================================================
## Puppet Server integration with gitlab server

### Puppet Server Setup
sudo -i

/opt/puppetlabs/puppet/bin/gem install r10k
/opt/puppetlabs/puppet/bin/r10k

mkdir /etc/puppetlabs/puppetserver/ssh
ssh-keygen -m PEM -t rsa -b 2048 -P '' -f /etc/puppetlabs/puppetserver/ssh/id-control_repo.rsa
chown puppet:puppet /etc/puppetlabs/puppetserver/ssh/id-control_repo.rsa*
chown puppet:puppet -R /etc/puppetlabs/puppetserver/
chmod 755 /etc/puppetlabs/puppetserver/ssh/
chmod 600 /etc/puppetlabs/puppetserver/ssh/id-control_repo.rsa*

mkdir /etc/puppetlabs/r10k

vi /etc/puppetlabs/r10k/r10k.yaml

```
cachedir: '/var/cache/r10k'

sources:
  control-repo:
    remote: 'git@git.stopitsomemore.com:puppet/control-repo.git'
    basedir: '/etc/puppetlabs/code/environments'
```

chown puppet:puppet -R /etc/puppetlabs/code/
mkdir /var/cache/r10k
chown puppet:puppet -R /var/cache/r10k/

mkdir /opt/puppetlabs/server/data/puppetserver/.ssh

`vi /opt/puppetlabs/server/data/puppetserver/.ssh/config`

```
host git.stopitsomemore.com
 HostName git.stopitsomemore.com
 IdentityFile /etc/puppetlabs/puppetserver/ssh/id-control_repo.rsa
 User git
```

chmod 700 /opt/puppetlabs/server/data/puppetserver/.ssh
chmod 600 /opt/puppetlabs/server/data/puppetserver/.ssh/config
ssh git.stopitsomemore.com    --- generates the known_hosts file
cp /root/.ssh/known_hosts /opt/puppetlabs/server/data/puppetserver/.ssh/
chown puppet:puppet -R /opt/puppetlabs/server/data/puppetserver/

useradd --create-home --shell /bin/bash --user-group --password erijfEFSEF3554jfe gitlab-runner

`vi /home/gitlab-runner/puppet_deploy.sh`

```
sudo -n -H -u puppet bash -c "/opt/puppetlabs/puppet/bin/r10k deploy environment $1 --verbose --puppetfile"
```

```
    chown gitlab-runner:gitlab-runner -R /home/gitlab-runner/
    chmod +x /home/gitlab-runner/puppet_deploy.sh
```

### Git Server Setup
Control-repo \ Setings \ CI/CD, Deploy Keys from the public key: /etc/puppetlabs/puppetserver/ssh/id-control_repo.rsa.pub

