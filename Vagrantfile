Vagrant.configure("2") do |config|
  # Default box and key behaviour
  config.vm.box = "ubuntu/jammy64"
  # Do not replace Vagrant default private key (keeps deterministic SSH keys)
  config.ssh.insert_key = false

  nodes = [
    { name: "controller",  ip: "192.168.56.10", memory: 2048, cpus: 2, box: "ubuntu/jammy64" },
    { name: "linux-client", ip: "192.168.56.11", memory: 1024, cpus: 1, box: "ubuntu/jammy64" },
    { name: "windows-client", ip: "192.168.56.12", memory: 2048, cpus: 2, box: "gusztavvargadr/windows-10" }
  ]

  nodes.each do |n|
    config.vm.define n[:name] do |node|
      # use per-node box (windows-client will use a Windows box)
      node.vm.box = n[:box] || "ubuntu/jammy64"
      node.vm.hostname = n[:name]
      node.vm.network "private_network", ip: n[:ip]

      # If this is the Windows VM, use WinRM communicator
      if n[:name] == "windows-client"
        node.vm.communicator = "winrm"
        # do not create the default synced folder on Windows to avoid SMB complications
        # (controller will connect over WinRM)
        # You can enable a synced folder if needed.
      else
        # Keep project root synced into /vagrant for Linux guests
        node.vm.synced_folder ".", "/vagrant", type: "virtualbox"
      end

      # VirtualBox provider settings
      node.vm.provider "virtualbox" do |vb|
        vb.name = "ansible-#{n[:name]}"
        vb.memory = n[:memory]
        vb.cpus = n[:cpus]
      end

      # Keep project root synced into /vagrant
      node.vm.synced_folder ".", "/vagrant", type: "virtualbox"

      # For Linux guests add file+shell provisioning
      unless n[:name] == "windows-client"
        # Copy host public key (if exists) into guest and basic provisioning
        # The keys file path is relative to this Vagrantfile: ./keys/id_rsa.pub
        node.vm.provision "file", source: "keys/id_rsa.pub", destination: "/tmp/id_rsa.pub", run: "always"

        node.vm.provision "shell", inline: <<-SHELL
          set -e
          apt-get update -y
          DEBIAN_FRONTEND=noninteractive apt-get install -y python3 python3-apt openssh-server
          systemctl enable ssh || true

          if [ -f /tmp/id_rsa.pub ]; then
            mkdir -p /home/vagrant/.ssh
            cat /tmp/id_rsa.pub >> /home/vagrant/.ssh/authorized_keys
            chown -R vagrant:vagrant /home/vagrant/.ssh
            chmod 600 /home/vagrant/.ssh/authorized_keys
          fi
        SHELL
      end
    end
  end
end
