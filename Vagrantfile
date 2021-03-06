# -*- mode: ruby -*-
# vi: set ft=ruby :

# Vagrant multi machine configuration

require 'yaml'
config_yml = YAML.load_file(File.open(__dir__ + '/vagrant-config.yml'))

NON_ROOT_USER = 'vagrant'.freeze
SWAPSIZE = 1000

# # Clone ansible-bootstrap repository
# system("
#   if [ #{ARGV[0]} = 'up' ]; then
#     test -d roles || mkdir -p roles
#     if [ ! -d roles/ansible-debops ]; then
#       echo 'Cloning ansible-debops role'
#       git clone https://github.com/debops/ansible-debops roles/ansible-debops
#     else
#       echo 'Updating ansible-debops role'
#       cd roles/ansible-debops && git pull ; cd - >/dev/null
#     fi
#   fi
# ")

Vagrant.configure(2) do |config|
  # set auto update to false if you do NOT want to check the correct additions version when booting this machine
  # config.vbguest.auto_update = true

  config_yml[:vms].each do |name, settings|
    # use the config key as the vm identifier
    config.vm.define name.to_s, autostart: true, primary: true do |vm_config|
      config.ssh.insert_key = false
      vm_config.vm.usable_port_range = (2200..2250)

      # This will be applied to all vms

      # set auto_update to false, if you do NOT want to check the correct
      # additions version when booting this machine
      vm_config.vbguest.auto_update = false

      # Ubuntu
      vm_config.vm.box = settings[:box]

      # Vagrant can share the source directory using rsync, NFS, or SSHFS (with the vagrant-sshfs
      # plugin). Consult the Vagrant documentation if you do not want to use SSHFS.
      # Get's honored normally
      # vm_config.vm.synced_folder '.', '/vagrant', disabled: true
      # # But not the centos box
      # vm_config.vm.synced_folder '.', '/home/vagrant/sync', disabled: true

      # TODO: Should we try this???
      # Change the permission of files and directories
      # so that nosetests runs without extra arguments.
      config.vm.synced_folder '.', '/vagrant', mount_options: ['dmode=775,fmode=664']
      vm_config.vm.synced_folder '.', '/shared', type: 'nfs'

      # assign an ip address in the hosts network
      vm_config.vm.network 'private_network', ip: settings[:ip]

      vm_config.vm.hostname = settings[:hostname]

      config.vm.provider 'virtualbox' do |v|
        # make sure that the name makes sense when seen in the vbox GUI
        v.name = settings[:hostname]

        # Be nice to our users.
        # v.customize ['modifyvm', :id, '--cpuexecutioncap', '50']
        v.customize ['modifyvm', :id, '--memory', settings[:ram], '--cpus', settings[:cpu]]
        v.customize ['modifyvm', :id, '--natdnshostresolver1', 'on']
        v.customize ['modifyvm', :id, '--chipset', 'ich9']

        v.customize ['modifyvm', :id, '--ioapic', 'on'] # Bug 51473
        v.customize ['modifyvm', :id, '--natdnshostresolver1', 'on']
        v.customize ['modifyvm', :id, '--natdnsproxy1', 'on']
        # Prevent clock drift, see http://stackoverflow.com/a/19492466/323407
        v.customize ['guestproperty', 'set', :id, '/VirtualBox/GuestAdd/VBoxService/--timesync-set-threshold', 10_000]

        # v.customize ['modifyvm', :id, '--usb', 'on']
        # v.customize ['modifyvm', :id, '--audio', 'coreaudio', '--audiocontroller', 'ac97']
        v.customize ['modifyvm', :id, '--audio', 'none']

        # v.customize ["modifyvm", :id, "--natdnshostresolver2", "on"]
      end

      # If you want to create an array where each entry is a single word, you can use the %w{} syntax, which creates a word array:
      # However, notice that the %w{} method lets you skip the quotes and the commas.
      hostname_with_hyenalab_tld = "#{settings[:hostname]}.hyenalab.home"

      aliases = [hostname_with_hyenalab_tld, settings[:hostname]]

      if Vagrant.has_plugin?('vagrant-hostsupdater')
        vm_config.hostsupdater.aliases = aliases
      elsif Vagrant.has_plugin?('vagrant-hostmanager')
        vm_config.hostmanager.enabled = true
        vm_config.hostmanager.manage_host = true
        vm_config.hostmanager.manage_guests = true
        vm_config.hostmanager.ignore_private_ip = false
        vm_config.hostmanager.include_offline = true
        vm_config.hostmanager.aliases = aliases
      end

      # TODO: Get rid of /etc/hosts bash script command
      vm_config.vm.provision 'shell' do |s|
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

      # FIXME: DISABLED 11/30/2018 ... we don't want to do all of this stuff in shell scripts! Ansible should handle it all!
      # Enable provisioning with a shell script. Additional provisioners such as
      # Puppet, Chef, Ansible, Salt, and Docker are also available. Please see the
      # documentation for more information about their specific syntax and use.
      # vm_config.vm.provision 'shell' do |s|
      #   s.inline = <<-SHELL
      #     if [ -f /vagrant_bootstrap ]; then
      #       echo "vagrant_bootstrap EXISTS ALREADY"
      #       exit 0
      #     fi

      #     sudo apt-get update && sudo apt-get install python htop ncdu -y && sudo apt-get install -f

      #     apt-get update
      #     apt-get install -y \
      #       apt-transport-https \
      #       ca-certificates \
      #       curl \
      #       python3-pip \
      #       software-properties-common
      #     pip3 install virtualenv
      #     echo vm.max_map_count=262144 > /etc/sysctl.d/vm_max_map_count.conf
      #     sysctl --system
      #     grep -qF "#{NON_ROOT_USER} - nofile 65536" /etc/security/limits.conf || echo "#{NON_ROOT_USER} - nofile 65536" >> /etc/security/limits.conf

      #     echo vm.max_map_count=262144 > /etc/sysctl.d/vm_max_map_count.conf
      #     sysctl --system
      #     grep -qF '#{NON_ROOT_USER} - nofile 65536' /etc/security/limits.conf || echo '#{NON_ROOT_USER} - nofile 65536' >> /etc/security/limits.conf
      #     grep -qF 'root - nofile 65536' /etc/security/limits.conf || echo 'root - nofile 65536' >> /etc/security/limits.conf

      #     # NOTE: Improving Performance on Low-Memory Linux VMs
      #     # NOTES: https://www.codero.com/knowledge-base/content/3/389/en/custom-swap-on-linux-virtual-machines.html
      #     # size of swapfile in megabytes
      #     swapsize=#{SWAPSIZE}

      #     # does the swap file already exist?
      #     grep -q "swapfile" /etc/fstab

      #     # if not then create it
      #     if [ $? -ne 0 ]; then
      #       echo 'swapfile not found. Adding swapfile.'
      #       fallocate -l ${swapsize}M /swapfile
      #       chmod 600 /swapfile
      #       mkswap /swapfile
      #       swapon /swapfile
      #       echo '/swapfile none swap defaults 0 0' >> /etc/fstab
      #     else
      #       echo 'swapfile found. No changes made.'
      #     fi

      #     # output results to terminal
      #     df -h
      #     cat /proc/swaps
      #     cat /proc/meminfo | grep Swap

      #     # https://www.codero.com/knowledge-base/content/3/388/en/improving-performance-on-low_memory-linux-vms.html
      #     echo vm.swappiness = 10 >> /etc/sysctl.d/30-vm-swappiness.conf
      #     echo vm.vfs_cache_pressure = 50 >> /etc/sysctl.d/30-vm-vfs_cache_pressure.conf
      #     sysctl -p

      #     DEBIAN_FRONTEND=noninteractive apt-get update; apt-get install -y \
      #     sudo \
      #     bash-completion \
      #     apt-file \
      #     autoconf \
      #     automake \
      #     gettext \
      #     yelp-tools \
      #     flex \
      #     bison \
      #     build-essential \
      #     ccache \
      #     curl \
      #     git \
      #     lcov \
      #     libbz2-dev \
      #     libffi-dev \
      #     libreadline-dev \
      #     libsqlite3-dev \
      #     libssl-dev \
      #     python3-pip \
      #     vim \
      #   ; \
      #         apt-get update \
      #   ; \
      #     DEBIAN_FRONTEND=noninteractive apt-get install -y python-six python-pip \
      #   ; \
      #         rm -rf /var/lib/apt/lists/*

      #   # FIXME: Get this into a role, systemctl 9/29/2018
      #   apt-get update
      #   apt-get install linux-headers-$(uname -r) -y
      #   sysctl net.ipv4.tcp_available_congestion_control
      #   echo net.core.default_qdisc=fq > /etc/sysctl.d/30-tcp_congestion_control.conf
      #   echo net.ipv4.tcp_congestion_control=bbr >> /etc/sysctl.d/30-tcp_congestion_control.conf
      #   sysctl -p

      #   touch /vagrant_bootstrap && \
      #   chown #{NON_ROOT_USER}:#{NON_ROOT_USER} /vagrant_bootstrap
      #   SHELL
      #   s.privileged = true
      # end

      # FIXME: Get this into a role, ansible install bcc 9/29/2018
      #   vm_config.vm.provision 'shell' do |s|
      #     s.inline = <<-SHELL
      #     apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys D4284CDD
      #     echo "deb https://repo.iovisor.org/apt/bionic bionic main" | tee /etc/apt/sources.list.d/iovisor.list
      #     apt-get update
      #     apt-get install bcc-tools libbcc-examples linux-headers-$(uname -r) -y
      #     SHELL
      #     s.privileged = true
      #   end

      # vm_config.vm.provision :ansible do |ansible|
      #   ansible.host_key_checking	= 'false'
      #   # Disable default limit to connect to all the machines
      #   ansible.limit = 'all'
      #   ansible.playbook = 'vagrant_playbook.yml'
      #   ansible.groups = config_yml[:groups]
      #   ansible.verbose = 'vvv'
      #   ansible.extra_vars = {
      #     deploy_env: 'vagrant'
      #   }
      #   # ansible.skip_tags = %w[bootstrap]
      #   ansible.raw_arguments = ['--forks=10']
      # end
    end
  end
end
