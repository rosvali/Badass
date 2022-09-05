CPU = 2
MEMORY = 4096
NAME = "badass"

Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/focal64"
  config.vm.box_url = "https://app.vagrantup.com/ubuntu/boxes/focal64"

  config.vm.provider "virtualbox" do |vb|
    vb.cpus = CPU
    vb.memory = MEMORY
    vb.name = NAME
    vb.gui = true
    vb.customize ['modifyvm', :id, '--clipboard-mode', 'bidirectional']
    vb.customize ['modifyvm', :id, '--draganddrop', 'bidirectional']
    vb.customize ["modifyvm", :id, "--vram", "128"]
  end

  config.vm.synced_folder ".", "/home/vagrant/Desktop/badass", type: "virtualbox"
  config.vm.provision "shell", name: "Setting up VM", privileged: false,  inline: <<-SHELL
    sudo apt-get update
    sudo apt-get install -y --no-install-recommends gdm3 ubuntu-desktop-minimal

    sudo add-apt-repository -y ppa:gns3/ppa
    # sudo rm /var/lib/dpkg/lock-frontend
    # sudo dpkg --configure -a
    # sudo apt-get install gnome-session-flashback

    # sudo apt-get install -y gns3-gui
    # sudo apt-get install -y docker.io
    # sudo usermod -aG docker "$USER"

    cd /tmp
    curl https://raw.githubusercontent.com/GNS3/gns3-server/master/scripts/remote-install.sh > gns3-remote-install.sh
    sudo bash gns3-remote-install.sh

    DEBIAN_FRONTEND=noninteractive sudo apt-get install -y gns3-gui
    # sudo systemctl disable gns3.service
    sudo usermod -aG docker "$USER"
    # sudo usermod -aG kvm,libvirt,docker,ubridge,wireshark "$USER"
    # gsettings set org.gnome.shell favorite-apps "$(gsettings get org.gnome.shell favorite-apps | sed s/.$//), 'gns3.desktop', 'wireshark.desktop', 'org.gnome.Terminal.desktop']"
  SHELL
end