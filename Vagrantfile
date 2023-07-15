# -*- mode: ruby -*-
# vi: set ft=ruby :

# Set LoadBalancer IP
MASTER_IP = "192.168.33.10"

Vagrant.configure("2") do |config|

  # Ubuntu VM with Nginx
  config.vm.define "nginx" do |nginx|
    nginx.vm.box = "generic/ubuntu1804"
    nginx.vm.network "private_network", ip: MASTER_IP
    nginx.vm.provider :vmware_desktop do |vmware|
      vmware.vmx["ethernet1.pcislotnumber"] = "33"
      vmware.vmx["memsize"] = "1024"
      vmware.vmx["numvcpus"] = "2"
    end
    nginx.vm.synced_folder __dir__, "/vagrant/nginx"
    nginx.vm.provision "file", source: "#{__dir__}/nginx/nginx.conf", destination: "/vagrant/nginx/nginx.conf"
    nginx.vm.provision "shell", inline: <<-SHELL
      sudo sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config 
      sudo systemctl restart sshd
      sudo ufw allow 6443/tcp
      sudo ufw reload
      sudo apt-get update
      sudo apt-get install -y nginx
      sudo mv /vagrant/nginx/nginx.conf /etc/nginx/nginx.conf
      sudo systemctl restart nginx
    SHELL
  end

  # CentOS VMs with K3s server
  (1..3).each do |i|
    config.vm.define "k3s-server-#{i}" do |server|
      server.vm.box = "generic/centos7"
      server.vm.network "private_network", ip: "192.168.33.#{10 + i}"
      server.vm.provider :vmware_desktop do |vmware|
        vmware.vmx["ethernet1.pcislotnumber"] = "33"
        vmware.vmx["memsize"] = "1024"
        vmware.vmx["numvcpus"] = "2"
      end
      server.vm.synced_folder __dir__, "/vagrant"
      server.vm.provision "shell", env: {"i" => i.to_s, "MASTER_IP" => MASTER_IP}, inline: <<-SHELL
        sudo sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config 
        sudo systemctl restart sshd
        sudo systemctl stop firewalld
        if [ "$i" -eq "1" ]; then
          curl -sfL https://get.k3s.io | sh -s - server --cluster-init --node-name k3s-server-$i
          TOKEN=$(sudo cat /var/lib/rancher/k3s/server/node-token)
          echo $TOKEN > /vagrant/token-$i.txt
        else
          while [ ! -f /vagrant/token-1.txt ]; do
            echo "Waiting for /vagrant/token-1.txt to be created..."
            sleep 5
          done
          TOKEN=$(cat /vagrant/token-1.txt)
          curl -sfL https://get.k3s.io | K3S_URL=https://$MASTER_IP:6443 K3S_TOKEN=$TOKEN INSTALL_K3S_EXEC="--node-name k3s-server-$i --tls-san 192.168.234.172 --tls-san 192.168.33.10" sh -
        fi
      SHELL
    end
  end

  # CentOS VM with K3s agent
  (1..2).each do |i|
    config.vm.define "k3s-agent-#{i}" do |agent|
      agent.vm.box = "generic/centos7"
      agent.vm.network "private_network", ip: "192.168.33.#{14 + i}"
      agent.vm.provider :vmware_desktop do |vmware|
        vmware.vmx["ethernet1.pcislotnumber"] = "33"
        vmware.vmx["memsize"] = "1024"
        vmware.vmx["numvcpus"] = "2"
      end
      agent.vm.synced_folder __dir__, "/vagrant/"
      agent.vm.provision "shell", env: {"MASTER_IP" => MASTER_IP}, inline: <<-SHELL
        sudo sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config 
        sudo systemctl restart sshd
        while [ ! -f /vagrant/token-1.txt ]; do
          echo "Waiting for /vagrant/token-1.txt to be created..."
          sleep 5
        done
        TOKEN=$(cat /vagrant/token-1.txt)
        curl -sfL https://get.k3s.io | K3S_URL=https://$MASTER_IP:6443 K3S_TOKEN=$TOKEN INSTALL_K3S_EXEC="--node-name k3s-agent-#{i}" sh -
        sudo ln -s /usr/local/bin/kubectl /usr/bin/
      SHELL
    end
  end
end