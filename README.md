# Install Puppet Server on Ubuntu

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

## Install foreman on puppetmaster server to manage agents on GUI.

echo "deb http://deb.theforeman.org/ focal 2.5" | sudo tee /etc/apt/sources.list.d/foreman.list
echo "deb http://deb.theforeman.org/ plugins 2.5" | sudo tee -a /etc/apt/sources.list.d/foreman.list
sudo apt-get -y install ca-certificates
wget -q https://deb.theforeman.org/pubkey.gpg -O- | sudo apt-key add -

sudo apt-get update && sudo apt-get -y install foreman-installer

sudo foreman-installer

# Install Puppet Agent on Debian / RedHat based OS
## Install Puppet Agent Ubuntu

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

## Install Puppet Agent CentOS

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

============================================================================================
# Puppet server module creation process

## Step1
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

## To validate the class file.
puppet parser validate manifests/install.pp

## To create init.pp manifest
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




============================================================================================
# Puppet Server integration with gitlab server

## Puppet Server Setup
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

chown gitlab-runner:gitlab-runner -R /home/gitlab-runner/
chmod +x /home/gitlab-runner/puppet_deploy.sh


## Git Server Setup
Control-repo \ Setings \ CI/CD, Deploy Keys from the public key: /etc/puppetlabs/puppetserver/ssh/id-control_repo.rsa.pub

