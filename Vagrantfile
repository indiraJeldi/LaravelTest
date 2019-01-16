# -*- mode: ruby -*-
# vi: set ft=ruby :

VAGRANTFILE_API_VERSION = '2'

@script = <<SCRIPT
# Install dependencies
apt-get update
apt-get install -y apache2 curl php5 php5-pgsql php5-curl php5-mcrypt php5-mysql php5-intl  php

# Configure Apache
echo "<VirtualHost *:80>

         ServerName larvelTest.vagrant
         ServerAlias larvelTest.vagrant

         DocumentRoot /var/www/MyQAApp/public
         SetEnv "APP_ENV" "vagrant"

       <Directory /var/www/MyQAApp/public>
          AllowOverride All
       </Directory>

       ErrorLog ${APACHE_LOG_DIR}/error.log
       CustomLog ${APACHE_LOG_DIR}/access.log combined

</VirtualHost>" > /etc/apache2/sites-available/vagrant.conf
a2enmod rewrite

ln -s /etc/apache2/sites-available/vagrant.conf /etc/apache2/sites-enabled/
rm -f /etc/apache2/sites-enabled/000-default.conf

service apache2 restart

# add test hosts
echo "
127.0.0.1 localhost larvelTest.vagrant
#10.238.135.101 pgsql-test pgsql-test.ideas-solutions.intern
" > /etc/hosts

if [ -e /usr/local/bin/composer ]; then
    /usr/local/bin/composer self-update
else
    curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer
fi

# Reset home directory of vagrant user
if ! grep -q "cd /var/www" /home/vagrant/.profile; then
    echo "cd /var/www" >> /home/vagrant/.profile
fi

#echo "** Run the following command to install dependencies, if you have not already:"
#echo "   vagrant ssh -c 'composer install'"
#echo "** Visit http://localhost:8095 in your browser for to view the application **"


SCRIPT

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.box = 'ubuntu/trusty64'
  config.vm.network "forwarded_port", guest: 80, host: 8095, host_ip: "127.0.0.1"
  config.vm.network "private_network", type: "dhcp"
  config.vm.network "private_network", ip: "192.168.56.1", virtualbox__intnet: true
  config.vm.synced_folder '.', '/var/www/'
  config.vm.provision 'shell', inline: @script

  config.vm.provider "virtualbox" do |vb|
    vb.customize ["modifyvm", :id, "--memory", "4028"]
    vb.customize ["modifyvm", :id, "--name", "larvelTest"]
  end
end