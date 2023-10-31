# Check for required plugins
required_plugins = %w[vagrant-reload vagrant-vbguest]
needs_restart = false

required_plugins.each do |plugin|
  unless Vagrant.has_plugin?(plugin)
    system("vagrant plugin install #{plugin}")
    needs_restart = true
  end
end

if needs_restart
  puts "Vagrant is restarting to load the new plugins..."
  exec("vagrant #{ARGV.join(' ')}")
end


Vagrant.configure("2") do |config|
  # vbguest
  config.vbguest.auto_update = true
  config.vbguest.installer_hooks[:before_start] = [
    "apt-get update",
    "apt-get install -y build-essential",
    "apt-get install -y linux-headers-$(uname -r)"
  ]

  config.vm.define "PICsimLab" do |picsimlab|
    picsimlab.vm.box = "bento/ubuntu-22.04"
    picsimlab.vm.hostname = "picsimlab"
    # Debugger TCP Port
    picsimlab.vm.network "forwarded_port", guest: 1234, host: 1234
    # Remote Control Interface
    picsimlab.vm.network "forwarded_port", guest: 5000, host: 5000

    config.vm.provider "virtualbox" do |vb|
      vb.name = "PICsimLab-vm"
      vb.gui = true
      vb.memory = "4096"
      vb.cpus = 2
      vb.customize ['modifyvm', :id, '--graphicscontroller', 'vmsvga']
      vb.customize ['modifyvm', :id, '--vram', '128']
      vb.customize ["modifyvm", :id, "--clipboard-mode", "bidirectional"]
      vb.customize ["modifyvm", :id, "--draganddrop", "bidirectional"]
    end

    config.vm.provision :shell, name: "Update and Upgrade", privileged: true, inline: <<-SHELL
      export DEBIAN_FRONTEND=noninteractive
      apt-get update
      apt-get upgrade -y
    SHELL

    config.vm.provision :shell, name: "Install Desktop Environment", privileged: true, inline: <<-SHELL
      export DEBIAN_FRONTEND=noninteractive
      apt-get install -y --no-install-recommends ubuntu-desktop
    SHELL

    config.vm.provision :shell, name: "Install PICsimLab", privileged: true, inline: <<-SHELL
      export DEBIAN_FRONTEND=noninteractive
      # requirements (ensurepip, xdg-icon-resource)
      apt-get install -y python3-venv xdg-utils

      # https://lcgamboa.github.io/picsimlab_docs/stable/Linux.html#x7-60002.3.1
      git clone --depth=1 https://github.com/lcgamboa/picsimlab.git
      cd picsimlab
      bscripts/build_all_and_install.sh

      # set serial communication
      # https://lcgamboa.github.io/picsimlab_docs/stable/SerialCommunication.html#x39-380005
      wget -O- http://www.piduino.org/piduino-key.asc | apt-key add -
      add-apt-repository -y 'deb http://apt.piduino.org jammy piduino'
      apt-get update
      apt-get install -y tty0tty-dkms
      usermod -a -G dialout vagrant
    SHELL

    # Reload provisioner to reboot the machine
    config.vm.provision :reload
  end
end
