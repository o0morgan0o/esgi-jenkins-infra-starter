Vagrant.configure("2") do |config|
    # Configuration de base pour toutes les machines
    config.vm.box = "debian/bookworm64"
  
    # Script commun pour installer Docker
    common_provision = <<-SCRIPT
      # ================================================== 
      # Install Docker and tools
      # ================================================== 
      apt-get update
      apt-get install -y apt-transport-https ca-certificates curl gnupg-agent software-properties-common tmux vim jq rsync
      curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add -
      add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu bionic stable"
      apt-get update
      apt-get install -y docker-ce docker-ce-cli containerd.io docker-compose
      usermod -aG docker vagrant

      # ==================================================
      # Installation zsh (better shell)
      # ==================================================
      echo "We are going to install zsh"
      sudo apt -y install zsh git
      echo "vagrant" | chsh -s /bin/zsh vagrant
      su - vagrant  -c  'echo "Y" | sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"'
      su - vagrant  -c "git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting"
      sed -i 's/^plugins=/#&/' /home/vagrant/.zshrc
      echo "plugins=(git  docker docker-compose colored-man-pages aliases copyfile  copypath dotenv zsh-syntax-highlighting jsontools)" >> /home/vagrant/.zshrc
      sed -i "s/^ZSH_THEME=.*/ZSH_THEME='agnoster'/g"  /home/vagrant/.zshrc

      # ================================================== 
      # Setting hosts
      # ================================================== 
      echo "192.168.56.150 jenkins.home.arpa" >> /etc/hosts
      echo "192.168.56.151 registry.home.arpa" >> /etc/hosts
      echo "192.168.56.152 prod.home.arpa" >> /etc/hosts
      echo "192.168.56.153 backup.home.arpa" >> /etc/hosts

      # ================================================== 
      # Allow push and pull to registry without https
      # ================================================== 
      echo '{ "insecure-registries" : ["registry.home.arpa:80"] }' > /etc/docker/daemon.json
      sudo systemctl restart docker
    SCRIPT
  
    # VM Jenkins
    config.vm.define "jenkins" do |jenkins|
      jenkins.vm.synced_folder "jenkins.home.arpa", "/home/vagrant/jenkins.home.arpa"
      jenkins.vm.synced_folder "shared", "/home/vagrant/shared"
      jenkins.vm.hostname = "jenkins.home.arpa"
      jenkins.vm.network "private_network", ip: "192.168.56.150"
      jenkins.vm.network "forwarded_port", guest: 80, host: 8150
      jenkins.vm.provider "virtualbox" do |vb|
        vb.name = "jenkins"
        vb.memory = 4096
        vb.cpus = 2
      end
      jenkins.vm.provision "shell", inline: common_provision
      jenkins.vm.provision "shell", inline: <<-SCRIPT
      echo 'cd /home/vagrant/jenkins.home.arpa' >> /home/vagrant/.bashrc
      echo 'cd /home/vagrant/jenkins.home.arpa' >> /home/vagrant/.zshrc
      SCRIPT
      jenkins.vm.provision "shell", inline: <<-SCRIPT
      # ==================================================
      # Prepare correct permissions on volume
      # ==================================================
      mkdir -p /home/vagrant/jenkins.home.arpa/vol-jenkins-data
      chown -R 1000:1000 /home/vagrant/jenkins.home.arpa/vol-jenkins-data
      SCRIPT
    end
  
    # VM Registry
    config.vm.define "registry" do |registry|
      registry.vm.synced_folder "registry.home.arpa", "/home/vagrant/registry.home.arpa"
      registry.vm.synced_folder "shared", "/home/vagrant/shared"
      registry.vm.hostname = "registry.home.arpa"
      registry.vm.network "private_network", ip: "192.168.56.151"
      registry.vm.network "forwarded_port", guest: 80, host: 8151
      registry.vm.provider "virtualbox" do |vb|
        vb.name = "registry"
        vb.memory = 1024
        vb.cpus = 1
      end
      registry.vm.provision "shell", inline: common_provision
      registry.vm.provision "shell", inline: <<-SCRIPT
      echo 'cd /home/vagrant/registry.home.arpa' >> /home/vagrant/.bashrc
      echo 'cd /home/vagrant/registry.home.arpa' >> /home/vagrant/.zshrc
      SCRIPT
    end
  
    # VM Production
    config.vm.define "prod" do |prod|
      prod.vm.synced_folder "prod.home.arpa", "/home/vagrant/prod.home.arpa"
      prod.vm.synced_folder "shared", "/home/vagrant/shared"
      prod.vm.hostname = "prod.home.arpa"
      prod.vm.network "private_network", ip: "192.168.56.152"
      prod.vm.network "forwarded_port", guest: 80, host: 8152
      prod.vm.provider "virtualbox" do |vb|
        vb.name = "prod"
        vb.memory = 2048
        vb.cpus = 1
      end
      prod.vm.provision "shell", inline: common_provision
      prod.vm.provision "shell", inline: <<-SCRIPT
      echo 'cd /home/vagrant/prod.home.arpa' >> /home/vagrant/.bashrc
      echo 'cd /home/vagrant/prod.home.arpa' >> /home/vagrant/.zshrc
      SCRIPT
    end

    # VM Backup
    config.vm.define "backup" do |backup|
      backup.vm.synced_folder "backup.home.arpa", "/home/vagrant/backup.home.arpa"
      backup.vm.synced_folder "shared", "/home/vagrant/shared"
      backup.vm.hostname = "backup.home.arpa"
      backup.vm.network "private_network", ip: "192.168.56.153"
      backup.vm.network "forwarded_port", guest: 80, host: 8153
      backup.vm.provider "virtualbox" do |vb|
        vb.name = "backup"
        vb.memory = 1024
        vb.cpus = 1
      end
      backup.vm.provision "shell", inline: common_provision
      backup.vm.provision "shell", inline: <<-SCRIPT
      echo 'cd /home/vagrant/backup.home.arpa' >> /home/vagrant/.bashrc
      echo 'cd /home/vagrant/backup.home.arpa' >> /home/vagrant/.zshrc
      SCRIPT
    end
  end