# frozen_string_literal: true

# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.

# vagrant plugin install vagrant-dnsmasq

NON_ROOT_USER = 'vagrant'.freeze

$instance_name_prefix = 'nfs'
$num_instances = 1 # Number of nodes, excluding master which is always created.
# $custom_networking_dnsDomain = ".demo.k8s.ap"  # put same value like custom.networking.dnsDomain in ansible's group_vars/all, BUT this time WITH THE DOT in front!
# E.g.  ".demo.k8s.ap",
# https://www.virtualbox.org/manual/ch08.html#vboxmanage-natnetwork
# def nat(config)
### Cannot be used, as the rest of vagrant commands fail...
#    config.vm.provider "virtualbox" do |v|
#      v.customize ["modifyvm", :id, "--nic1", "bridged", "--bridgeadapter", "enp3s0", "--nictype1", "virtio", "--macaddress1", "auto" ] #, "--nat-network2", "mybridgeinterface", "--nictype1", "virtio"] # 82540EM
#      v.customize ["modifyvm", :id, "--nic2", "nat", "--nictype2", "virtio"]
#    end
# end

Vagrant.configure(2) do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  # config.dnsmasq.domain = $custom_networking_dnsDomain
  # config.dnsmasq.domain = '.vbox.k8s.ap'

  # brew_prefix = `brew --prefix`.strip
  # config.dnsmasq.dnsmasqconf = brew_prefix + '/etc/dnsmasq.conf'

  # config.dnsmasq.reload_command = 'sudo brew services restart dnsmasq'

  # config.dnsmasq.keep_resolver_on_destroy = true

  # config.dnsmasq.resolver = '/etc/resolver'

  config.dnsmasq.disable = true

  # set auto_update to false, if you do NOT want to check the correct
  # additions version when booting this machine
  config.vbguest.auto_update = false

  # config.vm.box_check_update = "false"  # If there is no internet access to get new updates

  # config.vm.network "public_network", type: "dhcp", bridge: "enp3s0"
  # config.vm.network "public_network" #, :bridge => "enp3s0" #, mac: "auto" #, :adapter=>1 #, use_dhcp_assigned_default_route: true
  # config.ssh.port=22
  # config.vm.network "public_network", type: "dhcp", :bridge => "enp3s0"
  # config.vm.usable_port_range = (2000..2500)
  # config.vm.boot_timeout = 90
  # config.ssh.insert_key = false
  # config.ssh.username = "your_user"
  # config.ssh.password = "your_password"

  config.vm.provider 'virtualbox' do |vb|
    vb.gui = false # Set to true to view the window in graphical mode
    vb.memory = '2048' # "4096" #"3072" # 6144
    vb.cpus = 2
    # vb.customize ["storagectl", :id, "--name", "IDE Controller", "--remove"] # Make sure it does not use IDE
    # vb.customize ["storagectl", :id, "--name", "SATA Controller", "--add", "sata"]   # Make it use SATA: faster and less issues
    # optionally add: , "--hostiocache", "on", "--bootable", "on"] # like here: https://www.virtualbox.org/manual/ch08.html#vboxmanage-storagectl

    # vb.customize [
    #   "modifyvm", :id,
    #   "--memory", '2048',
    #   "--natdnshostresolver1", "on"
    # ]

    # vb.customize ["modifyvm", :id, "--natdnsproxy1", "on"]
  end

  #### CHOOSE DESIRED OS:
  # config.vm.box = "centos/7"
  # config.vm.box = "centos/atomic-host" # NEVER TESTED
  config.vm.box = 'ubuntu/xenial64'

  # NODES:
  (1..$num_instances).each do |i|
    config.vm.define vm_name = format('%s-worker-%02d%s', $instance_name_prefix, i, $custom_networking_dnsDomain) do |node|
      # node.vm.synced_folder ".vagrant", "/vagrant", type: "rsync" #, rsync__exclude: ".local_only" #rsync__include: ".vagrant/"
      # node.vm.box = "centos/7"
      # node.vm.box = "centos/atomic-host"
      node.vm.hostname = vm_name
      # node.ssh.host = vm_name
      # node.vm.provision "shell", inline: "echo hello from %s" % [node.vm.hostname]
      # node.vm.provision "shell" do |s|
      # s.path= "dockerize.sh"  # no longer required, handled by ansible
      # s.args= "node"
      # end

      # TODO: Get rid of /etc/hosts bash script command
      config.vm.provision 'shell' do |s|
        s.inline = <<-SHELL
            if [ -f /vagrant_bootstrap ]; then
              echo "vagrant_bootstrap EXISTS ALREADY"
              exit 0
            fi

            sudo apt-get -y update
            sudo apt-get -y install python-minimal python-apt
            HOSTNAME=`hostname`; sudo sed -ri \"/127\.0\.0\.1.*$HOSTNAME.*/d\" /etc/hosts

            cat /etc/hosts

            DEBIAN_FRONTEND=noninteractive apt-get update; apt-get install -y \
            sudo \
            bash-completion \
            curl \
            git \
            vim \
          ; \
                apt-get update \
          ; \
            DEBIAN_FRONTEND=noninteractive apt-get install -y python-six python-pip \
          ; \
          touch /vagrant_bootstrap && \
          chown #{NON_ROOT_USER}:#{NON_ROOT_USER} /vagrant_bootstrap
        SHELL
        s.privileged = true
      end # end - vm_config.vm.provision 'shell' do |s|

      config.dnsmasq.ip = proc do |guest_machine|
        guest_machine.communicate.sudo("ifconfig enp0s | grep 'inet addr' | cut -d ':' -f 2 | cut -d ' ' -f 1") do |_type, data|
          # return something like '192.168.59.100' or ['192.168.59.100', '192.168.59.103']
          config.dnsmasq.ip = data
        end
      end

      node.vm.provision 'shell', inline: <<-SHELL
       sudo cp -rf ~vagrant/.ssh ~root/ || true  # This will allow us to ssh into root with existing vagrant key
       sudo cp -rf ~ubuntu/.ssh ~root/ || true  # This will allow us to ssh into root with existing vagrant key
       #chmod 755 /vagrant/dockerize.sh
       #/vagrant/dockerize.sh
      SHELL
      # File.open("ssh_config", "w+") { |file| file.write("boo" ) }
    end
  end

  # MASTER:
  config.vm.define vm_name = format('%s-master-01%s', $instance_name_prefix, $custom_networking_dnsDomain), primary: true do |nfsmaster|
    # nfsmaster.vm.synced_folder ".vagrant", "/vagrant", type: "rsync" #, rsync__exclude: ".local_only" #rsync__include: ".vagrant/"
    # nfsmaster.vm.hostname = "#{nfsmaster}"
    # nfsmaster.vm.hostname = "%s" % [ nfsmaster ]
    nfsmaster.vm.hostname = vm_name
    # nfsmaster.ssh.host = vm_name
    # nfsmaster.vm.network "forwarded_port", guest: 80, host: 2080, auto_correct: true
    # nfsmaster.vm.network "forwarded_port", guest: 443, host: 2443, auto_correct: true

    # TODO: Get rid of /etc/hosts bash script command
    config.vm.provision 'shell' do |s|
      s.inline = <<-SHELL
          if [ -f /vagrant_bootstrap ]; then
            echo "vagrant_bootstrap EXISTS ALREADY"
            exit 0
          fi

          sudo apt-get -y update
          sudo apt-get -y install python-minimal python-apt
          HOSTNAME=`hostname`; sudo sed -ri \"/127\.0\.0\.1.*$HOSTNAME.*/d\" /etc/hosts

          cat /etc/hosts

          DEBIAN_FRONTEND=noninteractive apt-get update; apt-get install -y \
          sudo \
          bash-completion \
          curl \
          git \
          vim \
        ; \
              apt-get update \
        ; \
          DEBIAN_FRONTEND=noninteractive apt-get install -y python-six python-pip \
        ; \
        touch /vagrant_bootstrap && \
        chown #{NON_ROOT_USER}:#{NON_ROOT_USER} /vagrant_bootstrap
      SHELL
      s.privileged = true
    end # end - vm_config.vm.provision 'shell' do |s|

    config.dnsmasq.ip = proc do |guest_machine|
      guest_machine.communicate.sudo("ifconfig enp0s | grep 'inet addr' | cut -d ':' -f 2 | cut -d ' ' -f 1") do |_type, data|
        # return something like '192.168.59.100' or ['192.168.59.100', '192.168.59.103']
        config.dnsmasq.ip = data
      end
    end

    # nfsmaster.vm.provision :shell, inline: "echo hello from %s" % [nfsmaster.vm.hostname]
    # nfsmaster.vm.provision "shell" do |s|
    # s.path= "dockerize.sh"  # no longer required, handled by ansible
    # s.args= "master"
    # end

    nfsmaster.vm.provision 'shell', inline: <<-SHELL
     sudo cp -rf ~vagrant/.ssh ~root/ || true  # This will allow us to ssh into root with existing vagrant key
     sudo cp -rf ~ubuntu/.ssh ~root/ || true # This will allow us to ssh into root with existing vagrant key
     #chmod 755 /vagrant/dockerize.sh
     #/vagrant/dockerize.sh
     # curl -SL https://github.com/ReSearchITEng/kubeadm-playbook/archive/master.tar.gz | tar xvz # already in /vagrant
    SHELL
  end

  # Disable automatic box update checking. If you disable this, then
  # boxes will only be checked for updates when the user runs
  # `vagrant box outdated`. This is not recommended.
  # config.vm.box_check_update = false

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  # config.vm.network "forwarded_port", guest: 80, host: 8080

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  # config.vm.network "private_network", ip: "192.168.33.10"

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  # config.vm.network "public_network"

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  # config.vm.synced_folder "../data", "/vagrant_data"

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  #
  # config.vm.provider "virtualbox" do |vb|
  #   # Display the VirtualBox GUI when booting the machine
  #   vb.gui = true
  #
  #   # Customize the amount of memory on the VM:
  #   vb.memory = "1024"
  # end
  #
  # View the documentation for the provider you are using for more
  # information on available options.

  # Define a Vagrant Push strategy for pushing to Atlas. Other push strategies
  # such as FTP and Heroku are also available. See the documentation at
  # https://docs.vagrantup.com/v2/push/atlas.html for more information.
  # config.push.define "atlas" do |push|
  #   push.app = "YOUR_ATLAS_USERNAME/YOUR_APPLICATION_NAME"
  # end

  # Enable provisioning with a shell script. Additional provisioners such as
  # Puppet, Chef, Ansible, Salt, and Docker are also available. Please see the
  # documentation for more information about their specific syntax and use.
  # config.vm.provision "shell", inline: <<-SHELL
  #   sudo apt-get update
  #   sudo apt-get install -y apache2
  # SHELL
end
