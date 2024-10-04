# This environment variable is needed for WSL only
# when the Vagrant file is not directly under the user
# home directory
ENV['VAGRANT_WSL_WINDOWS_ACCESS_USER_HOME_PATH'] = ENV['PWD']
HOST_TIMEZONE = File.read('/etc/timezone').strip

Vagrant.configure("2") do |config|
  
  # Provision a Splunk Enterprise Server
	config.vm.define "splunksh" do |splunksh|
		splunksh.vm.box = "bento/ubuntu-22.04"
		splunksh.vm.box_version = "202407.23.0"
		#splunksh.vm.network "public_network", bridge: "", use_dhcp_assigned_default_route: true
		splunksh.vm.network "public_network", bridge: "", ip: "192.168.1.241"
		splunksh.vm.synced_folder '.', '/vagrant', disabled: true
		splunksh.vm.hostname = "splunksh"
		# Set up VirtualBox as the provider
		splunksh.vm.provider :virtualbox do |vb|
			vb.memory = 8192
			vb.cpus = 4
			# This names the VM in VirtualBox
			vb.name = "Splunk-SH"
		end
		splunksh.vm.provision "shell", inline: <<-SHELL
			sudo timedatectl set-timezone #{HOST_TIMEZONE}
			SPLUNK='splunk-9.3.1-0b8d769cb912-Linux-x86_64.tgz'
			cd /tmp
			echo "Downloading splunk..."
			wget --quiet -O $SPLUNK "https://download.splunk.com/products/splunk/releases/9.3.1/linux/$SPLUNK"
			echo "Untarring splunk into /opt..."
			sudo tar -xzf $SPLUNK -C /opt --checkpoint=1000 --checkpoint-action=. 2>&1
			cd /opt/splunk/bin
			sudo ./splunk enable boot-start -user root -systemd-managed 1 --accept-license --no-prompt \
        --answer-yes --seed-passwd asdf0987
			sudo systemctl enable Splunkd
			sudo systemctl start Splunkd
			sudo ./splunk add user brian -role User -password shiner123 -auth admin:asdf0987
			sudo ./splunk enable listen -port 9997 -auth admin:asdf0987
			sudo systemctl restart Splunkd
		SHELL
 end

  # Provision a Splunk Forwarder
	config.vm.define "splunkfwd" do |splunkfwd|
		splunkfwd.vm.box = "bento/ubuntu-22.04"
		splunkfwd.vm.box_version = "202407.23.0"
		#splunkfwd.vm.network "public_network", bridge: "", use_dhcp_assigned_default_route: true
		splunkfwd.vm.network "public_network", bridge: "", ip: "192.168.1.242"
		splunkfwd.vm.synced_folder '.', '/vagrant', disabled: true
		splunkfwd.vm.hostname = "splunkfwd"
		# Set up VirtualBox as the provider
		splunkfwd.vm.provider :virtualbox do |vb|
			vb.memory = 4096
			vb.cpus = 2
			# This names the VM in VirtualBox
			vb.name = "Splunk-FWD"
		end
		splunkfwd.vm.provision "shell", inline: <<-SHELL
			sudo timedatectl set-timezone #{HOST_TIMEZONE}
			SPLUNK='splunkforwarder-9.3.1-0b8d769cb912-Linux-x86_64.tgz'
			cd /tmp
			echo "Downloading splunk forwarder..."
			wget -O $SPLUNK "https://download.splunk.com/products/universalforwarder/releases/9.3.1/linux/$SPLUNK"
			echo "Untarring splunk into /opt..."
			sudo tar -xzf $SPLUNK -C /opt --checkpoint=1000 --checkpoint-action=. 2>&1
			sudo useradd -m splunkfwd
			sudo chown -R splunkfwd:splunkfwd /opt/splunkforwarder
			cd /opt/splunkforwarder/bin
			sudo ./splunk enable boot-start -user splunkfwd -systemd-managed 1 --accept-license --no-prompt \
        --answer-yes --seed-passwd asdf0987
			sudo systemctl enable SplunkForwarder
			sudo systemctl start SplunkForwarder
			sudo ./splunk add forward-server 192.168.1.241:9997
			sudo ./splunk add monitor /var/log
			sudo systemctl restart SplunkForwarder
		SHELL
  end
end
