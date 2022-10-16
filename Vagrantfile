$basic_script = <<SCRIPT
echo "192.168.99.100 jenkins.do1.lab jenkins" >> /etc/hosts
echo "192.168.99.101 docker.do1.lab docker" >> /etc/hosts

SCRIPT

$jenkins_script = <<SCRIPT
echo "* Installing Jenkins"
sudo wget https://pkg.jenkins.io/redhat/jenkins.repo -O /etc/yum.repos.d/jenkins.repo
sudo rpm --import https://pkg.jenkins.io/redhat/jenkins.io.key
sudo dnf makecache
sudo dnf install jenkins -y

echo "* Installing Java"
sudo dnf install java-17-openjdk -y

sudo systemctl start jenkins
sudo systemctl enable jenkins

echo "* Open port:8080, 80"
sudo firewall-cmd --permanent --add-port=8080/tcp
sudo firewall-cmd --permanent --add-port=80/tcp
sudo firewall-cmd --reload
echo "******************************"
echo "* Open http://localhost:8080 and paste the password below"
cat /var/lib/jenkins/secrets/initialAdminPassword
echo "******************************"
sleep 10s
SCRIPT

$docker_script = <<SCRIPT
#!/bin/bash

echo "* Add Docker repository ..."
dnf config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo

echo "* Install Docker ..."
dnf install -y docker-ce docker-ce-cli containerd.io
sudo dnf install docker-compose-plugin -y

echo "* Enable and start Docker ..."
systemctl enable docker
systemctl start docker

echo "* Add vagrant user to Docker group ..."
usermod -aG docker vagrant

echo "* Add vagrant user to Wheel group ..."
usermod -aG wheel vagrant
SCRIPT

Vagrant.configure(2) do |config|
  config.ssh.insert_key = false
  config.vm.synced_folder "D:\\dohw4", "/vagrant"

    (0..1).each do |i|
        config.vm.define "hw4-#{i}" do |worker|
            worker.vm.box="shekeriev/centos-stream-9"
            if i == 0 then
                worker.vm.hostname = "jenkins.do1.lab"
                worker.vm.network "forwarded_port", guest: 80, host: 8000
                worker.vm.network "forwarded_port", guest: 8080, host: 8080
                worker.vm.provision "shell", inline: $jenkins_script
            else
                worker.vm.hostname = "docker.do1.lab"
                worker.vm.provision "shell", inline: $docker_script
            end
            worker.vm.network "private_network", ip: "192.168.99.10#{i}"
            worker.vm.provision "shell", inline: $basic_script
            worker.vm.provider :virtualbox do |vb|
            vb.customize ["modifyvm", :id, "--memory", "3072"]
            end
        end
    end
end

