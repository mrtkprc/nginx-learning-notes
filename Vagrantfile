Vagrant.configure("2") do |config|
    config.vm.hostname = "nginx-handbook-box"
    config.vm.box = "ubuntu/focal64"
    config.vm.define "nginx-handbook-box"
    config.vm.network "private_network", ip: "192.168.20.20"
    config.vm.provider "virtualbox" do |vb|
      vb.cpus = 1
      vb.memory = "1024"
      vb.name = "nginx-handbook"
    end
end
