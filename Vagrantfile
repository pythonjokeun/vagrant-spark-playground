# -*- mode: ruby -*-
# vi: set ft=ruby :

BOX_IMAGE = "bento/ubuntu-16.04"

# modify this to define number of slaves
N_SLAVE = 2

Vagrant.configure("2") do |config|

  # define master VM
  config.vm.define "master" do |subconfig|
    subconfig.vm.box = BOX_IMAGE
    subconfig.vm.hostname = "master"
    subconfig.vm.network "private_network", ip: "10.0.0.10"
    subconfig.vm.network "forwarded_port", guest: 8080, host: 8080
    
    # uncomment and modify these lines below to customize master VM computing resource
    # subconfig.vm.provider "virtualbox" do |vbox|
    #   vbox.memory = 4096
    #   vbox.cpus = 2
    # end
  end

  # define slave VMs
  (1..N_SLAVE).each do |index|
    config.vm.define "slave0#{index}" do |subconfig|
      subconfig.vm.box = BOX_IMAGE
      subconfig.vm.hostname = "slave#{index}"
      subconfig.vm.network "private_network", ip: "10.0.0.#{index + 10}"
      subconfig.vm.network "forwarded_port", guest: 8081, host: 8081

      # uncomment and modify these lines below to customize slave VM computing resource
      # subconfig.vm.provider "virtualbox" do |vbox|
      #   vbox.memory = 4096
      #   vbox.cpus = 2
      # end
    end
  end

  # install Spark on all VMs
  config.vm.provision "shell", inline: <<-SHELL
    apt-get update
    apt-get install -y --no-install-recommends default-jdk
    wget https://www-eu.apache.org/dist/spark/spark-2.4.0/spark-2.4.0-bin-hadoop2.7.tgz
    tar xvf spark-2.4.0-bin-hadoop2.7.tgz
    mv spark-2.4.0-bin-hadoop2.7 spark
    rm -f spark-2.4.0-bin-hadoop2.7.tgz
  SHELL

  # run Spark on master VM as master
  config.vm.define "master" do |subconfig|
    subconfig.vm.provision :shell, inline: <<-SHELL
      mv spark/conf/spark-env.sh.template spark/conf/spark-env.sh
      echo "SPARK_MASTER_HOST=10.0.0.10" >> spark/conf/spark-env.sh 
    SHELL

    subconfig.trigger.after :up do |trigger|
      trigger.run_remote = {
        inline: <<-SHELL
          spark/sbin/stop-master.sh
          spark/conf/spark-env.sh 
          spark/sbin/start-master.sh  
        SHELL
      }
    end
  end

  # run Spark on slave VMs as slaves
  (1..N_SLAVE).each do |index|
    config.vm.define "slave0#{index}" do |subconfig|
      subconfig.trigger.after :up do |trigger|
        trigger.run_remote = {
          inline: <<-SHELL
            spark/sbin/stop-slave.sh
            spark/sbin/start-slave.sh spark://10.0.0.10:7077
          SHELL
        }
      end
    end
  end
end
