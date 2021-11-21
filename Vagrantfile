# -*- mode: ruby -*-
# vi: set ft=ruby :

# Maintainer:   jeffskinnerbox@yahoo.com / www.jeffskinnerbox.me
# Version:      0.0.3



servers=[
  {
    :define => "headless ubuntu",
    :hostname => "ubuntu-headless",  # set hostname
    :box => "ubuntu-headless",       # ubuntu 20.04 guest vm
    :disk => "50GB",
    :ram => 2048,
    :cpu => 1
  }
]

# Vagrantfile API/syntax version.  The "2" is the Vagrant configuration version.
VAGRANTFILE_API_VERSION = "2"

Vagrant.require_version ">= 2.2.6"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
    servers.each do |machine|
        config.vm.define machine[:define] do |node|
            # define the virtual machine
            node.vm.box = machine[:box]
            node.vm.hostname = machine[:hostname]
            node.disksize.size = machine[:disk]

            node.ssh.forward_agent = true        # if true, agent forwarding over SSH connections is enabled
            node.ssh.forward_x11 = true          # if true, X11 forwarding over SSH connections is enabled

            # set auto_update to false, if you do NOT want to check the correct
            # guest additions version when booting this machine
            if Vagrant.has_plugin?("vagrant-vbguest") then
                  node.vbguest.auto_update = false
            end

            # configure virtual machine
            node.vm.provider "virtualbox" do |vb|
                vb.gui = false                                             # true = GUI, false = headless
                vb.customize ["modifyvm", :id, "--cpus", machine[:cpu]]    # number of cpu allocated
                vb.customize ["modifyvm", :id, "--memory", machine[:ram]]  # ram memory allocated

                # enable USB and add a filter based on the desired device manufacturer / product
                #vb.customize ["modifyvm", :id, "--usb", "on"]           # usb 1.1 (CHCI) controller
                vb.customize ["modifyvm", :id, "--usbehci", "on"]       # usb 2.0 (EHCI) controller
                vb.customize ["modifyvm", :id, "--usbxhci", "on"]       # usb 3.0 (xHCI) controller
                vb.customize ['usbfilter', 'add', '0', '--target', :id,
                    '--name', 'QuickCam Orbit/Sphere AF',
                    '--vendorid', '0x046d',
                    '--productid', '0x0994']
                vb.customize ['usbfilter', 'add', '0', '--target', :id,
                    '--name', 'Adafruit Console Cable',
                    '--vendorid', '0x10c4',
                    '--productid', '0xea60']
            end

            # avoid the updating of guest additions if the user has the plugin installed (any version)
            # to update the plugin, use 'vagrant plugin update vagrant-vbguest'
            if Vagrant.has_plugin?("vagrant-vbguest")
                node.vbguest.auto_update = false
            end

            ## copy ssh key from the ansible control node to the ansible managed nodes
            #node.vm.provision "shell", name: "copy ssh key from the ansible control (as root)", run: "once" do |s|
                #ssh_pub_key = File.readlines("#{Dir.home}/.ssh/ansible.pub").first.strip
                #s.inline = <<-SHELL
                    #echo #{ssh_pub_key} >> /home/vagrant/.ssh/authorized_keys
                    #echo #{ssh_pub_key} >> /root/.ssh/authorized_keys
                #SHELL

            ## create a forwarded port mapping which allows access to a specific port
            #node.vm.network "forwarded_port", guest: 8888, host: 8888, host_ip: "127.0.0.1"  # port for jupyter notebook
            #node.vm.network "forwarded_port", guest: 1880, host: 1880, host_ip: "127.0.0.1"  # port for node-red
            #node.vm.network "forwarded_port", guest: 8086, host: 8086, host_ip: "127.0.0.1"  # port for influxdb
            #node.vm.network "forwarded_port", guest: 3000, host: 3000, host_ip: "127.0.0.1"  # port for grafana
            #node.vm.network "forwarded_port", guest: 8765, host: 8765, host_ip: "127.0.0.1"  # port for motioneye
            #node.vm.network "forwarded_port", guest: 8123, host: 8123, host_ip: "127.0.0.1"  # port for home assistant

            ## configure virtual network (private network, using static ip)
            #node.vm.network "private_network", ip: machine[:ip]            # static ip address

            # Create a public network, which generally matched to bridged network.
            # Bridged networks make the machine appear as another physical device on your network.
            $def_net = `ip route | grep -E "^default" | awk '{printf "%s", $5; exit 0}'`   # default network interface
            #config.vm.network "public_network", bridge: "#$def_net", ip: "192.168.1.218"   # static ip addess
            config.vm.network "public_network", bridge: "#$def_net"                        # DHCP assigned ip address

            # update linux packages currently within virtual machine
            node.vm.provision "shell", name: "update linux packages (as root)",
                run: "always", inline: "apt-get -y update && apt-get -y dist-upgrade"

            # create your linux system environment
            node.vm.provision "shell", name: "create your system environment (as root)",
                run: "once", path: "./scripts/sys-env.sh"

            # install networking tools used within virtual machine
            node.vm.provision "shell", name: "install networking tools used in labs (as root)",
                run: "once", path: "./scripts/net-tools.sh"

            # create your working environment for vagrant login
            node.vm.provision "shell", name: "create your working environment (as vagrant)",
                run: "once", path: "./scripts/login-env.sh"

            # install your c++ and x windows libraries
            node.vm.provision "shell", name: "install your development libraries (as root)",
                run: "once", path: "./scripts/dev-env.sh"

            # install your python tools and libraries
            node.vm.provision "shell", name: "install your python tools and libraries (as root)",
                run: "once", path: "./scripts/python-dev-env.sh"

            # reboot to make sure everything is set
            node.vm.provision "shell", name: "reboot to make sure everything is setup (as root)",
                run: "once", inline: "shutdown -r now"
        end
    end
end


