MASTERS = 1
NODES = 0
GLUSTER = 3
DISKS = 2

if ARGV.length == 2
  if ARGV[1] == "-f"
    puts "You must use 'vagrant #{ARGV[0]} <name>, [<name>] ...'."
    puts "Run 'vagrant status' to view VM names."
    exit 1
  end
end


Vagrant.configure(2) do |config|
  config.ssh.insert_key = false
  config.ssh.forward_agent = false
  config.hostmanager.enabled = true
  config.hostmanager.ignore_private_ip = false
  config.landrush.enabled = true
  config.landrush.host 'console.vagrant.test', 'master1.vagrant.test'
  config.landrush.host 'apps.vagrant.test', 'master1.vagrant.test'

  config.vm.box = "mansm/CentOS-7"

  config.vm.provider "virtualbox" do |v|
    v.linked_clone = true
  end

  config.vm.synced_folder ".", "/home/vagrant/sync", disabled: true

  config.vm.define "registry" do |registry|
    registry.vm.hostname = "registry.vagrant.test"
    registry.vm.network "private_network", ip: "192.168.77.11"
    disk = "registry-disk2.vmdk"
    registry.vm.provider "virtualbox" do |v|
      v.cpus = 2
      v.memory = 2048
      unless File.exist?(disk)
        v.customize ['createhd', '--filename', disk, '--variant', 'Standard', '--size', 40000]
      end
      v.customize ['storageattach', :id,  '--storagectl', 'SATA Controller', '--port', 2, '--device', 0, '--type', 'hdd', '--medium', disk]
      v.customize ["modifyvm", :id, "--nicpromisc2", "allow-all"]
    end
    registry.vm.provision :ansible do |ansible|
      ansible.playbook = "ansible/playbook.yml"
      ansible.become = true
      ansible.limit = "registry"
      ansible.verbose = "v"
      ansible.host_key_checking = false
      ansible.groups = {
              "glusterfs" => ["gluster[1:#{MASTERS}].vagrant.test"],
              "masters" => ["master[1:#{MASTERS}].vagrant.test"],
              "etcd" => ["gluster[1:#{MASTERS}].vagrant.test"],
              "nodes" => ["node[1:#{NODES}].vagrant.test"],
              "registries" => ["registry.vagrant.test"]
            }
    end
  end

  (1..MASTERS).each do |master_id|
    config.vm.define "master#{master_id}" do |master|
      master.vm.hostname = "master#{master_id}.vagrant.test"
      master.vm.network "private_network", ip: "192.168.77.#{20+master_id}"
      disk = "master#{master_id}-disk2.vmdk"
      master.vm.provider "virtualbox" do |v|
        v.cpus = 2
        v.memory = 2048
        unless File.exist?(disk)
          v.customize ['createhd', '--filename', disk, '--variant', 'Standard', '--size', 40000]
        end
        v.customize ['storageattach', :id,  '--storagectl', 'SATA Controller', '--port', 2, '--device', 0, '--type', 'hdd', '--medium', disk]
        v.customize ["modifyvm", :id, "--nicpromisc2", "allow-all"]
      end
      if master_id == MASTERS
        master.vm.provision :ansible do |ansible|
          ansible.playbook = "ansible/playbook.yml"
          ansible.become = true
          ansible.verbose = "v"
          ansible.host_key_checking = false
          ansible.limit = "masters"
          ansible.groups = {
            "masters" => ["master[1:#{MASTERS}].vagrant.test"],
            "etcd" => ["master[1:#{MASTERS}].vagrant.test"],
            "nodes" => ["node[1:#{NODES}].vagrant.test"]
          }
          #ansible.raw_arguments = ["--inventory-file=/Users/mansmatulewicz/Documents/projecten/vagrant-atomic/inventory"]
        end
      end
    end
  end
  (1..NODES).each do |node_id|
    config.vm.define "node#{node_id}" do |node|
      node.vm.hostname = "node#{node_id}.vagrant.test"
      node.vm.network "private_network", ip: "192.168.77.#{30+node_id}"
      disk = "node#{node_id}-disk2.vmdk"
      node.vm.provider "virtualbox" do |v|
        v.cpus = 2
        v.memory = 1024
        unless File.exist?(disk)
          v.customize ['createhd', '--filename', disk, '--variant', 'Standard', '--size', 40000]
        end
        v.customize ['storageattach', :id,  '--storagectl', 'SATA Controller', '--port', 2, '--device', 0, '--type', 'hdd', '--medium', disk]
        v.customize ["modifyvm", :id, "--nicpromisc2", "allow-all"]
      end
      if node_id == NODES
        node.vm.provision :ansible do |ansible|
          ansible.playbook = "ansible/playbook.yml"
          ansible.become = true
          ansible.verbose = "v"
          ansible.host_key_checking = false
          ansible.limit = "node[1:#{NODES}]"
          ansible.groups = {
            "masters" => ["master[1:#{MASTERS}].vagrant.test"],
            "etcd" => ["master[1:#{MASTERS}].vagrant.test"],
            "nodes" => ["node[1:#{NODES}].vagrant.test"]
          }
          ansible.raw_arguments = ["--inventory-file=/Users/mansmatulewicz/Documents/projecten/vagrant-atomic/inventory"]
        end
      end
    end
  end
  (1..GLUSTER).each do |gluster_id|
    config.vm.define "gluster#{gluster_id}" do |gluster|
      gluster.vm.hostname = "gluster#{gluster_id}.vagrant.test"
      gluster.vm.network "private_network", ip: "192.168.77.#{40+gluster_id}"
      gluster.vm.provider "virtualbox" do |v|
        v.cpus = 2
        v.memory = 2048
        unless File.exist?("gluster#{gluster_id}-disk2.vmdk")
          v.customize ['createhd', '--filename', "gluster#{gluster_id}-disk2.vmdk", '--variant', 'Standard', '--size', 40000]
        end
        v.customize ['storageattach', :id,  '--storagectl', 'SATA Controller', '--port', 1, '--device', 0, '--type', 'hdd', '--medium', "gluster#{gluster_id}-disk2.vmdk"]
        v.customize ["modifyvm", :id, "--nicpromisc2", "allow-all"]
        
        (3..DISKS+2).each do |d|
          unless File.exist?("gluster#{gluster_id}-disk#{d}.vmdk")
            v.customize ['createhd', '--filename', "gluster#{gluster_id}-disk#{d}.vmdk", '--variant', 'Standard', '--size', 10000]
          end
          v.customize ['storageattach', :id,  '--storagectl', 'SATA Controller', '--port', d, '--device', 0, '--type', 'hdd', '--medium', "gluster#{gluster_id}-disk#{d}.vmdk"]       
        end
        if gluster_id == GLUSTER
          gluster.vm.provision :ansible do |ansible|
            ansible.playbook = "ansible/playbook.yml"
            ansible.become = true
            ansible.verbose = "v"
            ansible.host_key_checking = false
            ansible.limit = "glusterfs"
            ansible.groups = {
              "glusterfs" => ["gluster[1:#{MASTERS}].vagrant.test"],
              "masters" => ["master[1:#{MASTERS}].vagrant.test"],
              "etcd" => ["gluster[1:#{MASTERS}].vagrant.test"],
              "nodes" => ["node[1:#{NODES}].vagrant.test"],
              "registries" => ["registry.vagrant.test"]
            }
            ansible.raw_arguments = ["--inventory-file=/Users/mansmatulewicz/Documents/projecten/vagrant-atomic/inventory"]
          end
        end
      end
    end
  end
end