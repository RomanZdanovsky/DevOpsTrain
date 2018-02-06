$num_nodes = 2

Vagrant.configure("2") do |config|

  config.vm.box = "bertvv/centos72"
  config.vm.box_check_update = false

  config.vm.provider "virtualbox" do |vb|
    vb.gui = false
    vb.memory = "1024"
    vb.cpus = "1"
  end

  config.vm.define "apache" do |apache|

    apache.vm.network "forwarded_port", guest: 80, host: 18080
    apache.vm.hostname = "apache"
    apache.vm.network "private_network", ip: "192.168.0.10"

    apache.vm.provision "shell", inline: <<-SHELL

      yum install httpd -y

      systemctl enable httpd

      systemctl start httpd

      firewall-cmd --zone=public --add-port=80/tcp --permanent

      systemctl restart firewalld

      cp /vagrant/mod_jk.so /etc/httpd/modules/

      touch /etc/httpd/conf/workers.properties

      echo "worker.list=lb" >> /etc/httpd/conf/workers.properties
      echo "worker.lb.type=lb" >> /etc/httpd/conf/workers.properties
      echo "worker.lb.balance_workers=tomcat1, tomcat2" >> /etc/httpd/conf/workers.properties
      echo "worker.tomcat1.host=192.168.0.11" >> /etc/httpd/conf/workers.properties
      echo "worker.tomcat1.port=8009" >> /etc/httpd/conf/workers.properties
      echo "worker.tomcat1.type=ajp13" >> /etc/httpd/conf/workers.properties
      echo "worker.tomcat2.host=192.168.0.12" >> /etc/httpd/conf/workers.properties
      echo "worker.tomcat2.port=8009" >> /etc/httpd/conf/workers.properties
      echo "worker.tomcat2.type=ajp13" >> /etc/httpd/conf/workers.properties

      echo "LoadModule jk_module modules/mod_jk.so" >> /etc/httpd/conf/httpd.conf
      echo "JkWorkersFile conf/workers.properties" >> /etc/httpd/conf/httpd.conf
      echo "JkShmFile /tmp/shm" >> /etc/httpd/conf/httpd.conf
      echo "JkLogFile logs/mod_jk.log" >> /etc/httpd/conf/httpd.conf
      echo "JkLogLevel info" >> /etc/httpd/conf/httpd.conf
      echo "JkMount /test* lb" >> /etc/httpd/conf/httpd.conf

      systemctl restart httpd

    SHELL
  end

  (1..$num_nodes).each do |i|
    config.vm.define "tomcat#{i}" do |tomcat|

      tomcat.vm.hostname = "tomcat#{i}"
      tomcat.vm.network "private_network", ip: "192.168.0.#{10+i}"

      tomcat.vm.provision "shell", inline: <<-SHELL

        yum install tomcat tomcat-webapps tomcat-admin-webapps -y

        systemctl enable tomcat

        systemctl start tomcat

        mkdir /usr/share/tomcat/webapps/test

        touch /usr/share/tomcat/webapps/test/index.html

        echo "tomcat#{i}" >> /usr/share/tomcat/webapps/test/index.html

        firewall-cmd --zone=public --add-port=8009/tcp --permanent

        systemctl restart firewalld

      SHELL
    end
  end
end
