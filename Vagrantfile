$registry_script = <<SCRIPT
docker pull registry:2
docker run -d -p 5000:5000 registry:2
SCRIPT

$manager_script = <<SCRIPT
echo -e '{\n    "insecure-registries" : ["10.100.199.199:5000"]\n}' > /tmp/daemon.json
sudo mv /tmp/daemon.json /etc/docker
sudo systemctl restart docker
echo Swarm Init...
docker swarm init --listen-addr 10.100.199.200:2377 --advertise-addr 10.100.199.200:2377
docker swarm join-token --quiet worker > /vagrant/worker_token
sudo docker run -it -d -p 5151:8080 -v /var/run/docker.sock:/var/run/docker.sock dockersamples/visualizer
sudo docker run -d -p 9000:9000 -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer
SCRIPT

$worker_script = <<SCRIPT
echo Swarm Join...
docker swarm join --token $(cat /vagrant/worker_token) 10.100.199.200:2377
SCRIPT

Vagrant.configure('2') do |config|

  vm_box = 'vm-docker'



  config.vm.define "registry" do |registry|
    registry.vm.box = vm_box
    registry.vm.network :private_network, ip: "10.100.199.199"
    registry.vm.network :forwarded_port, guest: 5000, host: 5000
    registry.ssh.password = "vagrant"
    registry.vm.hostname = "registry"
    registry.vm.synced_folder ".", "/vagrant"
    registry.vm.provision "shell", inline: $registry_script, privileged: true
    registry.vm.provider "virtualbox" do |vb|
      vb.name = "registry"
      vb.memory = "512"
    end
  end

  config.vm.define :manager, primary: true  do |manager|
    manager.vm.box = vm_box
    manager.vm.network :private_network, ip: "10.100.199.200"
    manager.vm.network :forwarded_port, guest: 8080, host: 8080
    manager.vm.network :forwarded_port, guest: 9000, host: 9000
    manager.vm.network :forwarded_port, guest: 5151, host: 5151
    manager.ssh.password = "vagrant"
    manager.vm.hostname = "manager"
    manager.vm.synced_folder ".", "/vagrant"
    manager.vm.provision "shell", inline: $manager_script, privileged: true
    manager.vm.provider "virtualbox" do |vb|
      vb.name = "manager"
      vb.memory = "1024"
    end
  end

  (1..1).each do |i|
    config.vm.define "worker0#{i}" do |worker|
      worker.vm.box = vm_box
      worker.vm.network :private_network, ip: "10.100.199.20#{i}"
      worker.ssh.password = "vagrant"
      worker.vm.hostname = "worker0#{i}"
      worker.vm.synced_folder ".", "/vagrant"
      worker.vm.provision "shell", inline: $worker_script, privileged: true
      worker.vm.provider "virtualbox" do |vb|
        vb.name = "worker0#{i}"
        vb.memory = "1024"
      end
    end
  end

end
