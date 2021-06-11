# Install Puppet Server on Ubuntu
=================================

  _sudo nano /etc/hosts_

    ```
    [puppet master ip] puppetmaster puppet
    [puppet client ip] puppetclient
    ```

  _wget https://apt.puppetlabs.com/puppet6-release-focal.deb_

  _sudo dpkg -i puppet6-release-focal.deb

  sudo apt-get update -y

  sudo apt-get install puppetserver pdk -y

  sudo nano /etc/default/puppetserver_

    ```
    JAVA_ARGS="-Xms1g -Xmx1g -Djruby.logger.class=com.puppetlabs.jruby_utils.jruby.Slf4jLogger"
    ```

  _sudo systemctl enable --now puppetserver

  sudo /opt/puppetlabs/bin/puppetserver ca list --all
  sudo /opt/puppetlabs/bin/puppetserver ca sign --all

  sudo /opt/puppetlabs/bin/puppetserver ca sign --certname <Agent Name>
  sudo /opt/puppetlabs/bin/puppetserver ca clean --certname <Agent Name>_

# Install Puppet Agent Ubuntu
=============================

  _wget https://apt.puppetlabs.com/puppet6-release-focal.deb

  sudo dpkg -i puppet6-release-focal.deb

  sudo apt-get update -y

  sudo apt-get install puppet-agent -y

  sudo nano /etc/puppetlabs/puppet/puppet.conf_

    ```
    [main]
    certname = puppetclient
    server = puppetmaster
    ```

  _sudo systemctl enable --now puppetserver

  sudo /opt/puppetlabs/bin/puppet agent --test_


# Install Puppet Agent CentOS
=============================

  _sudo rpm -Uvh https://yum.puppet.com/puppet6/puppet6-release-el-7.noarch.rpm

  sudo yum install -y puppet-agent

  sudo nano /etc/puppetlabs/puppet/puppet.conf_

    '''
    [main]
    certname = puppetclient
    server = puppetmaster
    '''
  
  _sudo systemctl enable --now puppetserver

  sudo /opt/puppetlabs/bin/puppet agent --test_

# Module creation process
=========================
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

# to validate the class file.
puppet parser validate manifests/install.pp

# to create init.pp manifest
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

`cd /etc/puppetlabs/code/environment/production/manifests`

`nano site.pp`

```
node puppetagent1.test {
  class { 'nginx': }
}
```

