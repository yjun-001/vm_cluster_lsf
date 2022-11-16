# -*- mode: ruby -*-
# vi: set ft=ruby :
=begin
  author:  Jun
  version: 1.01
  usage: vagrant up
=end

def Get_Hosts_from_Inventory(inventory_File)
  @host_data = {}
  section = nil
  value = nil
  File.readlines(inventory_File).each do |line|
    line = line.chomp
    unless (/^\#/.match(line))    # skip line start with #, as comment lines
      if (line =~ /^([^=]+?)\s+([^=]+?)\s*=\s*(.*?)\s*$/)
        name = $1
        section = $2
        value = $3
        @host_data[name]=value
      end
      if (line =~ /^\[(.*)\]\s*$/)   # skip [] sections
         break
      end
    end
  end
  @host_data
end

def Set_Host_Public_SSHkey(machine, ssh_pub_key)
  machine.vm.provision "shell", inline: <<-SHELL
    if grep -sq "#{ssh_pub_key}" /home/vagrant/.ssh/authorized_keys; then
      echo "SSH keys already provisioned."
      exit 0; 
    fi
    echo "SSH key provisioning."
    mkdir -p /home/vagrant/.ssh/
    touch /home/vagrant/.ssh/authorized_keys
    echo #{ssh_pub_key} >> /home/vagrant/.ssh/authorized_keys
    echo #{ssh_pub_key} > /home/vagrant/.ssh/id_rsa.pub
    chmod 644 /home/vagrant/.ssh/id_rsa.pub
  SHELL
end

def Set_Host_Private_SSHkey(machine, ssh_prv_key)
  machine.vm.provision "shell", inline: <<-SHELL
    echo "#{ssh_prv_key}" > /home/vagrant/.ssh/id_rsa
    chmod 600 /home/vagrant/.ssh/id_rsa
    chown -R vagrant:vagrant /home/vagrant
  SHELL
end

Vagrant.configure("2") do |config|
  config.vm.box = "generic/ubuntu2004"
  #config.vm.hostname = "hpc-node"

  ssh_prv_key = File.read("#{Dir.home}/.ssh/id_rsa")
  ssh_pub_key = File.readlines("#{Dir.home}/.ssh/id_rsa.pub").first.strip
  host_port_count = 2500
  hosts = Get_Hosts_from_Inventory("#{Dir.pwd}/ansible/inventory/hosts")
  hosts.each do | node, ip |
    #puts node, ip
    #vgnode = ( node != 'master' ) ? node : "master-vg"
    #config.vm.define vgnode do | machine |
    config.vm.define node do | machine |
      machine.vm.hostname = "#{node}"
      #machine.vm.disk :disk, size: "3GB", primary: true
      machine.vm.network "forwarded_port", guest: 22, host: host_port_count, id: "ssh", auto_correct: false
      host_port_count += 1
      machine.vm.network "private_network", ip: "#{ip}"
      machine.vm.synced_folder 'ansible', '/vagrant', id: "vagrant-root", disabled: false, mount_options: ["dmode=775"]
      Set_Host_Public_SSHkey(machine, ssh_pub_key)

      if node == 'master'
        Set_Host_Private_SSHkey(machine, ssh_prv_key)

        # fix ubuntu ERROR: '~ansible' by install manually
        machine.vm.provision "shell", inline: <<-SHELL
          apt-get update
          apt-get install -y ansible 
        SHELL

        #machine.vm.synced_folder 'ansible', '/vagrant', id: "vagrant-root", disabled: false, mount_options: ["dmode=775"]
        #machine.vm.provision "file", source: "./ansible.cfg", destination: "/home/vagrant/.ansible.cfg"
        machine.vm.provision "ansible_local" do |ansible|
          ansible.become   = true
          ansible.limit    = "all" # or only "nodes" group, etc.
          ansible.playbook = "provision.yaml"
          ansible.inventory_path = "/vagrant/inventory"
          # ansible.verbose = true
          ansible.install = true
          #vagrant_synced_folder_default_type = ""
        end
      end
    end
  end
end