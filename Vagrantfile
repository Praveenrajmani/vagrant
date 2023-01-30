Vagrant.configure("2") do |config|
  config.vm.define "master" do |master|
    master.vm.box_download_insecure = true    
    master.vm.box = "generic/ubuntu2204"
    master.vm.network "private_network", ip: "100.0.0.1"
    master.vm.hostname = "master"
    master.vm.provider "libvirt" do |v|
      v.memory = 2048
      v.cpus = 2
    end
  end
  config.vm.define "node1" do |worker|
    worker.vm.box_download_insecure = true 
    worker.vm.box = "generic/ubuntu2204"
    worker.vm.network "private_network", ip: "100.0.0.2"
    worker.vm.hostname = "node1"
    worker.vm.provider "libvirt" do |v|
      v.memory = 1024
      v.cpus = 1
    end
  end
  config.vm.define "node2" do |worker|
    worker.vm.box_download_insecure = true 
    worker.vm.box = "generic/ubuntu2204"
    worker.vm.network "private_network", ip: "100.0.0.3"
    worker.vm.hostname = "node2"
    worker.vm.provider "libvirt" do |v|
      v.memory = 1024
      v.cpus = 1
    end
  end
  config.vm.define "node3" do |worker|
    worker.vm.box_download_insecure = true 
    worker.vm.box = "generic/ubuntu2204"
    worker.vm.network "private_network", ip: "100.0.0.4"
    worker.vm.hostname = "node3"
    worker.vm.provider "libvirt" do |v|
      v.memory = 1024
      v.cpus = 1
    end
  end
  config.vm.define "node4" do |worker|
    worker.vm.box_download_insecure = true 
    worker.vm.box = "generic/ubuntu2204"
    worker.vm.network "private_network", ip: "100.0.0.5"
    worker.vm.hostname = "node4"
    worker.vm.provider "libvirt" do |v|
      v.memory = 1024
      v.cpus = 1
    end
  end

end
