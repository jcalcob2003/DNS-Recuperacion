Vagrant.configure("2") do |config|
    # Configuración de DNSA (servidor maestro)
    config.vm.define "dnsa" do |dnsa|
      dnsa.vm.box = "ubuntu/bionic64"
      dnsa.vm.network "private_network", ip: "192.168.57.10"
      dnsa.vm.hostname = "dnsa"
      dnsa.vm.provision "shell", inline: <<-SHELL
        sudo apt update
        sudo apt install -y bind9 bind9utils bind9-doc
      SHELL
    end
  
    # Configuración de DNSB (servidor esclavo)
    config.vm.define "dnsb" do |dnsb|
      dnsb.vm.box = "ubuntu/bionic64"
      dnsb.vm.network "private_network", ip: "192.168.57.11"
      dnsb.vm.hostname = "dnsb"
      dnsb.vm.provision "shell", inline: <<-SHELL
        sudo apt update
        sudo apt install -y bind9 bind9utils bind9-doc
      SHELL
    end
  end