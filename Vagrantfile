# -*- mode: ruby -*-
# vi: set ft=ruby :

VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  os = "ubuntu/focal64"
  network_ip = "192.168.80"

  config.vm.define :master, primary: true do |jenkins_config|
    jenkins_config.vm.provider "virtualbox" do |vb|
        vb.memory = "2048"
        vb.cpus = 2
        vb.name = "jenkins-server"
    end

    jenkins_config.vm.box = "#{os}"
    jenkins_config.vm.host_name = "jenkins-server"
    jenkins_config.vm.network "private_network", ip: "#{network_ip}.10"

    jenkins_config.vm.provision "shell", inline: <<-SHELL
      apt-get update
      apt-get -y upgrade && apt-get -y dist-upgrade
      apt-get -y autoremove && apt-get -y autoclean
      apt-get -y install apt-transport-https ca-certificates gnupg-agent software-properties-common
      apt-get -y install default-jre
      wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
      sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
      apt-get update
      apt-get -y install jenkins
    SHELL
  end

  # Build Agents
  [
    ["agent1",    "#{network_ip}.11",    "1024",    os ],
    ["agent2",    "#{network_ip}.12",    "1024",    os ],
  ].each do |vmname,ip,mem,os|
    config.vm.define "#{vmname}" do |agent_config|
      agent_config.vm.provider "virtualbox" do |vb|
          vb.memory = "#{mem}"
          vb.cpus = 1
          vb.name = "#{vmname}"
      end

      agent_config.vm.box = "#{os}"
      agent_config.vm.hostname = "#{vmname}"
      agent_config.vm.network "private_network", ip: "#{ip}"

      agent_config.vm.provision "shell", inline: <<-SHELL
        apt-get update
        apt-get -y upgrade && apt-get -y dist-upgrade
        apt-get -y autoremove && apt-get -y autoclean
        apt-get -y install apt-transport-https ca-certificates gnupg-agent software-properties-common
      SHELL
    end
  end
end
