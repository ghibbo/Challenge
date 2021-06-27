Vagrant.configure("2") do |config|
  
  number_of_machines = 3
  
  box_name = "centos/8"

  base_ip = 100
  base_ip_addresses = "192.168.10"

  ip_addresses = (1..number_of_machines).map{ |i| "#{base_ip_addresses}.#{base_ip + i}" }

  config.vm.provider "virtualbox" do |v|
    v.memory = 2048
  end

  (1..number_of_machines).each do |i|
    config.vm.define "centos_vm#{i}" do |box|
      box.vm.box = box_name
      box.vm.network "private_network", ip: "#{ip_addresses[i-1]}"
      box.vm.hostname = "centos-vm#{i}"
      box.vm.provision "ansible", playbook: "main.yml"
    end
  end

end
