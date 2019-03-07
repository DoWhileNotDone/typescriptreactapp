VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

  config.vm.provider "virtualbox" do |v|
    v.memory = 2048
  end

  config.vm.box = "bento/ubuntu-18.04"
  config.vm.network "private_network", ip: "192.168.50.67"

  #config.vm.synced_folder ".", "/vagrant", type: "rsync", rsync_auto: true, rsync_exclude: [".git/", "node_modules/"]

  # Update apt packages
  config.vm.provision "shell", name: "apt", inline: <<-SHELL
    export DEBIAN_FRONTEND=noninteractive
    apt-get update && apt-get upgrade
    packagelist=(
      libssl1.0-dev
      libreadline-dev
      libyaml-dev
      libxml2-dev
      libxslt1-dev
      libnss3
      libx11-dev
      software-properties-common
      wget
      unzip
      curl
      ant
    )
    apt-get install -y ${packagelist[@]}
  SHELL

  #Install node/nvm/gulp
  config.vm.provision "shell", name: "install nvm and node", privileged: false, inline: <<-SHELL
    export DEBIAN_FRONTEND=noninteractive
    cd && curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.33.11/install.sh | bash
    export NVM_DIR="$HOME/.nvm"
    [ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"  # This loads nvm
    [ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"  # This loads nvm bash_completion
    nvm install node
    nvm use node
    npm install -g webpack webpack-cli
  SHELL

  # Install Yarn
  config.vm.provision "shell", name: "add yarn", inline: <<-SHELL

    YN_REPO_APT_SOURCE=/etc/apt/sources.list.d/yarn.list
    if [ ! -f "$YN_REPO_APT_SOURCE" ]
    then
      curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | sudo apt-key add -
      echo "deb https://dl.yarnpkg.com/debian/ stable main" | sudo tee /etc/apt/sources.list.d/yarn.list
    fi

    # https://github.com/yarnpkg/yarn/issues/2821
    apt-get purge -y cmdtest

    apt-get update && apt-get install -y  --no-install-recommends yarn

  SHELL

  config.vm.provision "shell", name: "install apache", inline: <<-'SHELL'
    apt-get install -y apache2
    #Allow port 80 through the firewall
    ufw allow 'Apache'
  SHELL

  # Update Apache config and restart
  config.vm.provision "shell", name: "configure apache", inline: <<-'SHELL'

    # Symlink DocumentRoot o \Vagrant\Publics
    ln -s /vagrant/dist /var/www/html/DocumentRoot

    sed -i -e "s/DocumentRoot \/var\/www\/html/DocumentRoot \/var\/www\/html\/DocumentRoot/" /etc/apache2/sites-enabled/000-default.conf
    sed -i -e "s/AllowOverride None/AllowOverride All/" /etc/apache2/apache2.conf

    a2enmod rewrite
    apachectl restart
    # Make sure Apache also runs after vagrant reload
    systemctl enable apache2
  SHELL

  config.vm.post_up_message = <<MESSAGE

   You are now up and running.

   This is the >> TypeScript React App Development Server

   The URL is 192.168.50.67

MESSAGE

end
