# # -*- mode: ruby -*-
# # vi: set ft=ruby :

# Vagrant.configure("2") do |config|

#     # Ubuntu VM with Nginx
#     config.vm.define "nginx" do |nginx|
#       nginx.vm.box = "generic/ubuntu1804"
#       nginx.vm.network "private_network", ip: "192.168.33.10"
#       nginx.vm.provider :vmware_desktop do |vmware|
#         vmware.vmx["ethernet1.pcislotnumber"] = "33"
#         vmware.vmx["memsize"] = "1024"
#         vmware.vmx["numvcpus"] = "2"
#       end
#       nginx.vm.synced_folder __dir__, "/vagrant/"
#       nginx.vm.provision "file", source: "#{__dir__}/nginx/nginx.conf", destination: "/vagrant/nginx.conf"
#       nginx.vm.provision "shell", inline: <<-SHELL
#         sudo sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config 
#         sudo systemctl restart sshd
#         sudo apt-get update
#         sudo apt-get install -y nginx
#         sudo mv /vagrant/nginx.conf /etc/nginx/nginx.conf
#         sudo systemctl restart nginx
#       SHELL
#     end
  
#     # CentOS VMs with K3s server
#     (1..3).each do |i|
#       config.vm.define "k3s-server-#{i}" do |server|
#         server.vm.box = "generic/centos7"
#         server.vm.network "private_network", ip: "192.168.33.#{10 + i}"
#         server.vm.provider :vmware_desktop do |vmware|
#             vmware.vmx["ethernet1.pcislotnumber"] = "33"
#             vmware.vmx["memsize"] = "1024"
#             vmware.vmx["numvcpus"] = "2"
#         end
#         server.vm.synced_folder __dir__, "/vagrant/"
#         server.vm.provision "shell", inline: <<-SHELL
#         sudo sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config 
#         sudo systemctl restart sshd
#         curl -sfL https://get.k3s.io | sh -s - server --cluster-init
#         TOKEN=$(sudo cat /var/lib/rancher/k3s/server/node-token)
#         sudo ln -s /usr/local/bin/kubectl /usr/bin/
#         if [ -z "$TOKEN" ]; then
#             echo "Failed to get K3s token"
#             exit 1
#         fi
#         echo $TOKEN > /vagrant/token-#{i}.txt
#         if [ $? -ne 0 ]; then
#             echo "Failed to write K3s token to file"
#             exit 1
#         fi
#         if [ $i -gt 1 ]; then
#           MASTER_IP="192.168.33.11"
#           curl -sfL https://get.k3s.io | K3S_URL=https://$MASTER_IP:6443 K3S_TOKEN=$TOKEN sh -s - server
#         fi
#         SHELL
#       end

#       # CentOS VM with K3s agent
#       config.vm.define "k3s-agent" do |agent|
#           agent.vm.box = "generic/centos7"
#           agent.vm.network "private_network", ip: "192.168.33.14"
#           agent.vm.provider :vmware_desktop do |vmware|
#             vmware.vmx["ethernet1.pcislotnumber"] = "33"
#             vmware.vmx["memsize"] = "1024"
#             vmware.vmx["numvcpus"] = "2"
#           end
#           agent.vm.synced_folder __dir__, "/vagrant/"
#           agent.vm.provision "shell", inline: <<-SHELL
#             sudo sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config 
#             sudo systemctl restart sshd
#             MASTER_IP="192.168.33.11"
#             while [ ! -f /vagrant/token-1.txt ]; do
#               echo "Waiting for /vagrant/token-1.txt to be created..."
#               sleep 5
#             done
#             TOKEN=$(cat /vagrant/token-1.txt)
#             curl -sfL https://get.k3s.io | K3S_URL=https://$MASTER_IP:6443 K3S_TOKEN=$TOKEN sh -s - agent
#             sudo ln -s /usr/local/bin/kubectl /usr/bin/
#           SHELL
#         end
# end

# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|

  # Ubuntu VM with Nginx
  config.vm.define "nginx" do |nginx|
    nginx.vm.box = "generic/ubuntu1804"
    nginx.vm.network "private_network", ip: "192.168.33.10"
    nginx.vm.provider :vmware_desktop do |vmware|
      vmware.vmx["ethernet1.pcislotnumber"] = "33"
      vmware.vmx["memsize"] = "1024"
      vmware.vmx["numvcpus"] = "2"
    end
    nginx.vm.synced_folder __dir__, "/vagrant/"
    nginx.vm.provision "file", source: "#{__dir__}/nginx/nginx.conf", destination: "/vagrant/nginx.conf"
    nginx.vm.provision "shell", inline: <<-SHELL
      sudo sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config 
      sudo systemctl restart sshd
      sudo apt-get update
      sudo apt-get install -y nginx
      sudo mv /vagrant/nginx.conf /etc/nginx/nginx.conf
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
      server.vm.synced_folder __dir__, "/vagrant/"
      server.vm.provision "shell", inline: <<-SHELL
        sudo sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config 
        sudo systemctl restart sshd
        curl -sfL https://get.k3s.io | sh -s - server --cluster-init
        TOKEN=$(sudo cat /var/lib/rancher/k3s/server/node-token)
        sudo ln -s /usr/local/bin/kubectl /usr/bin/
        if [ -z "$TOKEN" ]; then
          echo "Failed to get K3s token"
          exit 1
        fi
        echo $TOKEN > /vagrant/token-#{i}.txt
        if [ $? -ne 0 ]; then
          echo "Failed to write K3s token to file"
          exit 1
        fi
        if [ -n "$i" ] && [ "$i" -gt 1 ]; then
          MASTER_IP="192.168.33.11"
          curl -sfL https://get.k3s.io | K3S_URL=https://$MASTER_IP:6443 K3S_TOKEN=$TOKEN sh -s - server
        fi
      SHELL
    end
  end

  # CentOS VM with K3s agent
  config.vm.define "k3s-agent" do |agent|
    agent.vm.box = "generic/centos7"
    agent.vm.network "private_network", ip: "192.168.33.14"
    agent.vm.provider :vmware_desktop do |vmware|
      vmware.vmx["ethernet1.pcislotnumber"] = "33"
      vmware.vmx["memsize"] = "1024"
      vmware.vmx["numvcpus"] = "2"
    end
    agent.vm.synced_folder __dir__, "/vagrant/"
    agent.vm.provision "shell", inline: <<-SHELL
      sudo sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config 
      sudo systemctl restart sshd
      MASTER_IP="192.168.33.11"
      while [ ! -f /vagrant/token-1.txt ]; do
        echo "Waiting for /vagrant/token-1.txt to be created..."
        sleep 5
      done
      TOKEN=$(cat /vagrant/token-1.txt)
      curl -sfL https://get.k3s.io | K3S_URL=https://$MASTER_IP:6443 K3S_TOKEN=$TOKEN sh -s - agent
      sudo ln -s /usr/local/bin/kubectl /usr/bin/
    SHELL
  end
end
