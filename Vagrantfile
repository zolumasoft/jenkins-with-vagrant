# -*- mode: ruby -*-
# vi: set ft=ruby :

VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  os = "ubuntu/focal64"
  network_ip = "192.168.80"

  config.vm.define :jenkins, primary: true do |jenkins_config|
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
      apt-get -y install python3 python3-venv
      apt-get -y install default-jre
      wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
      sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
      apt-get update
      apt-get -y install jenkins
    SHELL
  end

  # Build Agents
  [
    ["agent1",    "#{network_ip}.11",    "2048",    os],
    ["agent2",    "#{network_ip}.12",    "2048",    os],
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
        apt-get -y install python3 python3-venv
        apt-get -y install default-jre
        adduser --disabled-login --gecos 'Jenkins' jenkins
        curl -o docker-key -fsSL https://download.docker.com/linux/ubuntu/gpg
        apt-key add docker-key > /dev/null 2>&1
        rm docker-key
        add-apt-repository -y "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
        apt-get update
        apt-get -y install docker-ce docker-ce-cli containerd.io
        usermod -aG docker vagrant
        usermod -aG docker jenkins
        if [ ! -d /home/jenkins/jenkins_agent ] && [ ! -d /home/jenkins/.ssh ]; then
          su - jenkins -c "mkdir /home/jenkins/jenkins_agent /home/jenkins/.ssh && chmod 600 /home/jenkins/.ssh"
          su - jenkins -c "touch /home/jenkins/.ssh/authorized_keys"
        fi
      SHELL
    end
  end
end
