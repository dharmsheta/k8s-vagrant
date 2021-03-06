# -*- mode: ruby -*-
# vi: set ft=ruby :

# Install a kube cluster using kubeadm:
# http://kubernetes.io/docs/getting-started-guides/kubeadm/

### configuration parameters
### Box parameters
box_base = "ubuntu/xenial64"
box_cpu_count = 1
box_ram_mb = "2048"
### kubeadm parameters
master_ip = "192.168.77.10"
join_token = "7d0576.ee0f7f72653463dd"
worker_count = 3
### This can break compatibilty, use with care. 
### Always make sure that kubeadm's version supports k8s version
### This is only for the control plane components, 
### kubelet and kubeadm will always be the latest available version
kubernetes_version = "v1.9.3"

Vagrant.require_version ">= 2.0.0"

Vagrant.configure("2") do |config|
  config.vm.box = box_base
  config.vm.box_check_update = false
  config.vm.provision "shell", path: "install.sh"

  # use cache plugin if available
  # https://github.com/fgrehm/vagrant-cachier
  if Vagrant.has_plugin?("vagrant-cachier")
    config.cache.scope = :box
  end

  config.vm.provider "virtualbox" do |vb|
    vb.cpus = box_cpu_count
    vb.memory = box_ram_mb
  end

  # master node configuration
  config.vm.define "kube-master", primary: true do |master|
    master.vm.hostname = "kube-master"
    master.vm.network "private_network", ip: master_ip

    master.vm.provision "shell" do |s|
      s.inline = "sed 's/127.0.0.1.*'$2'/'$1' '$2'/' -i /etc/hosts"
      s.args = [master_ip, master.vm.hostname]
    end

    master.vm.provision "shell" do |s|
      s.path = "master-setup.sh"
      s.privileged = false
      s.args = [master_ip, join_token, kubernetes_version]
    end

    master.vm.post_up_message = "Done. don't forget to modify the $KUBECONFIG env var to include the cluster conf file located in this folder"

  end
  
  # worker nodes configuration
  (1..worker_count).each do |i|
    config.vm.define "kube-worker#{i}" do |worker|
      worker_ip = master_ip.split('.').tap{|arr| arr[-1] = arr[-1].to_i + i}.join('.')

      worker.vm.hostname = "kube-worker#{i}"
      worker.vm.network "private_network", ip: worker_ip
            
      worker.vm.provision "shell" do |s|
        s.inline = "sed 's/127.0.0.1.*'$2'/'$1' '$2'/' -i /etc/hosts"
        s.args = [worker_ip, worker.vm.hostname]
      end

      worker.vm.provision "shell" do |s|
        s.path = "worker-setup.sh"
        s.args = [master_ip, join_token]
      end
    end
  end
end