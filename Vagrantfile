Vagrant.configure("2") do |config|
    servers=[
        {
          :hostname => "most1",
          :ip => "172.18.1.13",
          :name => 'most1',
        }
      ]

    servers.each do |machine|
        config.vm.define machine[:hostname] do |node|
            node.vm.box = "ubuntu/jammy64"
            node.vm.hostname = machine[:hostname]
			node.disksize.size = '15GB'
            node.vm.network :private_network, ip: machine[:ip]
			node.vm.network "forwarded_port", guest: 80, host: 80
			node.vm.network "forwarded_port", guest: 10050, host: 10050
			node.vm.network "forwarded_port", guest: 80, host: 8080
			node.vm.network "forwarded_port", guest: 8080, host: 80
            node.vm.provision "shell", inline: <<-SHELL
                sudo apt update && sudo apt upgrade -y
				sudo apt dist-upgrade -y
				sudo apt autoremove -y
				sudo apt install update-manager-core -y
            SHELL

            node.vm.provider :virtualbox do |vb|
                vb.customize ["modifyvm", :id, "--memory", 2048, "--cpus", 2]
                vb.name = machine[:name]
            end
                end
    end
end