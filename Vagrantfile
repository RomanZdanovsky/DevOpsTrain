Vagrant.configure("2") do |config|

  config.vm.box = "bertvv/centos72"
  config.vm.box_check_update = false

  config.vm.provider "virtualbox" do |vb|
     vb.gui = false
     vb.memory = "512"
     vb.cpus = "1"
  end

  config.vm.define "server1" do |server1|

     server1.vm.hostname = "server1"
     server1.vm.network "private_network", ip: "192.168.0.10"

     server1.vm.provision "shell", inline: <<-SHELL

         sudo echo "192.168.0.11 server2" >> /etc/hosts

         sudo yum install -y git

         git clone https://github.com/RomanZdanovsky/DevOpsTrain

         export GIT_DISCOVERY_ACROSS_FILESYSTEM=1

         cd DevOpsTrain

         git checkout task2

         cat greeting

     SHELL
  end

  config.vm.define "server2" do |server2|

     server2.vm.hostname = "server2"
     server2.vm.network "private_network", ip: "192.168.0.11"

     server2.vm.provision "shell", inline: <<-SHELL
         sudo echo "192.168.0.10 server1" >> /etc/hosts
     SHELL
  end
end
