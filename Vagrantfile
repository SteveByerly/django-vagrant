# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure("2") do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://atlas.hashicorp.com/search.
  config.vm.box = "ubuntu/trusty64"

  # Disable automatic box update checking. If you disable this, then
  # boxes will only be checked for updates when the user runs
  # `vagrant box outdated`. This is not recommended.
  # config.vm.box_check_update = false

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  config.vm.network "forwarded_port", guest: 80, host: 8080
  config.vm.network "forwarded_port", guest: 8000, host: 8001

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  # config.vm.network "private_network", ip: "192.168.33.10"

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  # config.vm.network "public_network"

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  config.vm.synced_folder ".", "/home/vagrant/django-vagrant-bootstrap/"

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  #
  config.vm.provider "virtualbox" do |vb|
    vb.memory = "1024"
    vb.name = "django-vagrant-bootstrap"
  end
  #
  # View the documentation for the provider you are using for more
  # information on available options.

  # Define a Vagrant Push strategy for pushing to Atlas. Other push strategies
  # such as FTP and Heroku are also available. See the documentation at
  # https://docs.vagrantup.com/v2/push/atlas.html for more information.
  # config.push.define "atlas" do |push|
  #   push.app = "YOUR_ATLAS_USERNAME/YOUR_APPLICATION_NAME"
  # end

  # Enable provisioning with a shell script. Additional provisioners such as
  # Puppet, Chef, Ansible, Salt, and Docker are also available. Please see the
  # documentation for more information about their specific syntax and use.
  config.vm.provision "shell", inline: <<-SHELL

    base="/home/vagrant/django-vagrant-bootstrap"
    django_base="$base/example-project"
    venv_base="/home/vagrant/.virtualenvs/example_project"
    
    echo "##### INSTALLING APT REQUIREMENTS #####"
    apt-get install -y python-dev python-virtualenv nginx

    echo "##### INSTALLING uWSGI GLOBALLY #####"
    pip install uwsgi

    echo "##### SETTING UP VIRTUALENV #####"
    virtualenv $venv_base
    echo "source $venv_base/bin/activate" >> /home/vagrant/.bashrc
    source $venv_base/bin/activate

    echo "##### INSTALLING PIP REQUIREMENTS #####"
    cd $django_base
    pip install -r requirements.txt

    echo "##### SETTING UP uWSGI #####"
    if [ ! -d /run/uwsgi ]
      then
        echo "# creating uwsgi socket directory #"
        mkdir /run/uwsgi
    fi
    chown vagrant:www-data /run/uwsgi

    if [ ! -d /var/log/uwsgi ]
      then
        echo "# creating uwsgi log directory #"
        mkdir /var/log/uwsgi
    fi
    chown vagrant:www-data /var/log/uwsgi

    echo "##### SETTING UP uWSGI EMPEROR MODE #####"
    if [ ! -d /etc/uwsgi ]
      then
        echo "# creating uwsgi sites directory #"
        mkdir /etc/uwsgi
        mkdir /etc/uwsgi/sites
    fi

    ln -sf $base/uwsgi/site.ini /etc/uwsgi/sites/
    
    echo "##### SETTING UP uWSGI SERVICE #####"
    if [ -f /etc/init/uwsgi.conf ]
      then
        echo "# removing existing uWSGI conf file #"
        rm /etc/init/uwsgi.conf
    fi

    cp $base/uwsgi/uwsgi.conf /etc/init/uwsgi.conf

    echo "##### SETTING UP NGINX #####"
    if [ -a /etc/nginx/sites-enabled/default ]
      then
        echo "# disabling default nginx conf #"
        rm /etc/nginx/sites-enabled/default
    fi

    if [ -a /etc/nginx/sites-available/site.conf ]
      then
        echo "# removing existing nginx conf #"
        rm /etc/nginx/sites-available/site.conf
    fi

    cp $base/nginx/site.conf /etc/nginx/sites-available/site.conf
    ln -sf /etc/nginx/sites-available/site.conf /etc/nginx/sites-enabled/site.conf

    echo "##### STARTING uWSGI & NGINX SERVICES #####"
    service uwsgi restart
    service nginx restart

  SHELL
end
