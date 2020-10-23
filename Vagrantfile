# -*- mode: ruby -*-
# vi: set ft=ruby :

vars = {
  'DOCKER_CE_VERSION' => ''
}

module OS
  def OS.windows?
    (/cygwin|mswin|mingw|bccwin|wince|emx/ =~ RUBY_PLATFORM) != nil
  end

  def OS.mac?
    (/darwin/ =~ RUBY_PLATFORM) != nil
  end

  def OS.unix?
    !OS.windows?
  end

  def OS.linux?
    OS.unix? and not OS.mac?
  end
end

Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/bionic64"
  config.vm.box_check_update = false

  config.vm.hostname = "mongo-rs0"

  # always use Vagrants insecure key
  config.ssh.insert_key = false
  config.ssh.username = 'vagrant'

  config.vm.network "forwarded_port", guest: 27017, host: 27017
  # config.vm.network "forwarded_port", guest: 80, host: 8080, host_ip: "127.0.0.1"

  config.vm.network "private_network", ip: "172.30.1.15", auto_config: true
  #config.vm.network "public_network"

  config.vm.synced_folder ".", "/vagrant", disabled: false

  config.vm.provider "virtualbox" do |vb|
    if OS.windows?
      # VBoxManage setextradata global "VBoxInternal/NEM/UseRing0Runloop" 0
      # VBoxManage.exe setextradata global "VBoxInternal/NEM/UseRing0Runloop" 0
      vb.customize ["setextradata", :id, "VBoxInternal/NEM/UseRing0Runloop", "0"]
    end
    vb.gui = false
    # Customize the amount of memory on the VM:
    vb.memory = "2048"
    vb.name = "mongo-rs0"
  end
  #
  # View the documentation for the provider you are using for more
  # information on available options.

  # Enable provisioning with a shell script. Additional provisioners such as
  # Puppet, Chef, Ansible, Salt, and Docker are also available. Please see the
  # documentation for more information about their specific syntax and use.
  config.vm.provision "shell", env: vars, privileged: true, inline: <<-SHELL
     apt-get update
     apt-get upgrade -y
     apt-get install -y apt-transport-https ca-certificates curl software-properties-common
     curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add -
     add-apt-repository -y "deb [arch=amd64] https://download.docker.com/linux/ubuntu bionic stable"
     apt-get update
     apt-cache policy docker-ce
     apt-get install -y docker-ce
     systemctl enable docker
     systemctl start docker
     systemctl status docker
     apt-get install -y openssh-server
     systemctl enable ssh
     systemctl start ssh
     systemctl status ssh

     curl -L "https://github.com/docker/compose/releases/download/1.27.4/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
     chmod +x /usr/local/bin/docker-compose
  SHELL
  config.vm.provision "shell", privileged: false, inline: <<-SHELL
    cd /vagrant
    sudo docker-compose up -d
    while ! nc -z localhost 27017; do sleep 1; done
    sudo docker exec -it localmongo1 mongo --eval 'rs.initiate({ _id: "rs0", members: [{_id: 0, host: "mongo-rs0:27017"}]})'
    sudo docker exec -it localmongo1 mongo --eval 'rs.status()'
  SHELL
end
