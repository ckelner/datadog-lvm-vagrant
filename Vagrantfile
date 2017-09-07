# -*- mode: ruby -*-
# vi: set ft=ruby :
# This script will bring up a vagrant with an unmounted volume
# And test that the datadog agent and report metrics on this volume
# You will need to set DD_API_KEY when you run vagrant, i.e.
# `DD_API_KEY=<YOURAPIKEYHERE> vagrant up`

Vagrant.configure("2") do |config|
  # Image to use
  config.vm.box = "ubuntu/precise64"
  config.vm.box_url = "http://files.vagrantup.com/precise64.box"

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  config.vm.provider "virtualbox" do |vb|
    vb.memory = "1024"
    file_to_disk = './tmp/disk.vdi'
    unless File.exist?(file_to_disk)
      vb.customize ['createhd', '--filename', file_to_disk, '--size', 500]
    end
    vb.customize ['storageattach', :id, '--storagectl', 'SATA Controller', '--port', 1, '--device', 0, '--type', 'hdd', '--medium', file_to_disk]
  end

  # Enable provisioning with a shell script. Additional provisioners such as
  # Puppet, Chef, Ansible, Salt, and Docker are also available. Please see the
  # documentation for more information about their specific syntax and use.
  config.vm.provision "shell", inline: <<-SHELL
    echo "Provisioning..."
    echo "------------------------------------------"
    echo "Setting up LVM..."
    sudo pvcreate /dev/sdb
    sudo pvscan
    sudo pvdisplay
    sudo vgcreate vol_grp1 /dev/sdb
    sudo vgdisplay
    sudo lvcreate -L 400 -n logical_vol1 vol_grp1
    sudo mkfs.ext3 /dev/vol_grp1/logical_vol1
    sudo vgchange -ay
    sudo vgscan
    sudo lsblk -o NAME,FSTYPE,SIZE,MOUNTPOINT,LABEL
    echo "------------------------------------------"
    echo "Setting up Datadog..."
    # Get the API Key from the host environment
    APIKEY=#{ENV['DD_API_KEY']}
    # NOTE: Change you values below if you want something different
    TAGS='name:lvm-test, role:lvm-test'
    HOSTNAME='lvm-test-kelner'
    # Re-check to see if able to load the DD API Key
    if [[ $APIKEY != "" && ! -z $APIKEY ]]; then
      sudo apt-get update
      sudo apt-get install apt-transport-https
      sudo sh -c "echo 'deb https://apt.datadoghq.com/ stable main' > /etc/apt/sources.list.d/datadog.list"
      sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys C7A7DA52
      sudo apt-get update
      sudo apt-get install datadog-agent
      # Hacky but it works...
      sudo sh -c "sed 's/api_key:.*/api_key: $APIKEY/' /etc/dd-agent/datadog.conf.example > /etc/dd-agent/datadog.conf"
      sudo sh -c "sed 's/# tags:.*/tags: $TAGS/' /etc/dd-agent/datadog.conf > /etc/dd-agent/tmp.conf"
      sudo sh -c "sed 's/# hostname:.*/tags: $HOSTNAME/' /etc/dd-agent/tmp.conf > /etc/dd-agent/temp.conf"
      sudo sh -c "mv /etc/dd-agent/temp.conf /etc/dd-agent/datadog.conf"
      sudo /etc/init.d/datadog-agent start
    fi
  SHELL
end
