Vagrant.configure("2") do |config|


  #lista di macchine che verranno create:
  
  servers=[
    
    #sul nodo puppet eseguo update e installo pacchetto puppet
    #imposto la share condivisa con il sistema host (la mia macchina Windows)
    {
      :hostname => "rocky1.puppet.vm",
	    :box => "generic/rocky8",
      :private_address => '192.168.56.10', #rete privata per far funzionare la comunicazione tra le macchine
      :shared_dir_host => "./data",
      :shared_dir_guest => "/home/vagrant/data",
      :script => <<-SCRIPT 
      echo I am provisioning...
      date > /etc/vagrant_provisioned_at && sudo yum update && 
      sudo dnf -y install https://yum.puppet.com/puppet-release-el-8.noarch.rpm && sudo dnf update -y && sudo dnf install puppet-agent -y &&
      sudo systemctl enable --now puppet && echo \"[main]
      ssldir = /var/lib/puppet/ssl
      vardir = /var/lib/puppet
      cadir = /var/lib/puppet/ssl/ca
      dns_alt_names = puppet
      [agent]
      server=puppetmaster-ipadress
      ca_server=puppetmaster-ipadress\"  >> /etc/puppetlabs/puppet/puppet.conf
      SCRIPT
    },
    
    # sul master puppet eseguo update e installo pacchetto puppet. per far funzionare la comunicazione tra le macchine su utilizza una rete privata. 
    # sempre per far funzionare la comunicazione, sul master si disattiva e stoppa il servizio firewalld
    # installo puppetserver e puppet
    # eseguo yum update
    # cambio della memoria allocata al puppet server, di default sono 2 giga, sono impostati 512 mega, riduciamo per evitare l'out of memory
    # avvio il servizio puppetserver
    # abilito il servizio puppetserver
    # inserisco l'ip del server con associato 'puppet' nel file hosts
    {
      :hostname => "master2.puppet.vm",
	    :box => "generic/centos8",
      :private_address => '192.168.56.11', #rete privata per far funzionare la comunicazione tra le macchine
      :shared_dir_host => "./master_code",
      :shared_dir_guest => "/etc/puppetlabs/",
      :script => <<-SCRIPT 
      echo I am provisioning...
      date > /etc/vagrant_provisioned_at && sudo yum update -y && 
      rpm -Uvh https://yum.puppet.com/puppet6-release-el-8.noarch.rpm && yum install puppetserver git -y && sudo systemctl stop firewalld.service && systemctl disable firewalld.service &&
      sed -i 's/Xms2g/Xms512m/g' /etc/sysconfig/puppetserver && sed -i 's/Xmx2g/Xmx512m/g' /etc/sysconfig/puppetserver && systemctl start puppetserver &&
      systemctl enable puppetserver && echo 192.168.56.11 master2.puppet.vm puppet >> /etc/hosts
      SCRIPT
  
    },
	

  	{
     :hostname => "rocky2.puppet.vm",
	 :box => "generic/rocky8",  
     :script => "",
     :private_address => '192.168.56.13',
     :shared_dir_host => "./data",
     :shared_dir_guest => "/home/vagrant/data"
    },
	

    # per un nodo Puppet, scarico Puppet e installo l'agent
    # inserisco l'hostname/IP del master in /etc/hosts
    {
      :hostname => "clientc.puppet.vm",
      :box => "generic/centos8", 
      :private_address => '192.168.56.14',
      :shared_dir_host => "./data",
      :shared_dir_guest => "/home/vagrant/data",
      :script => <<-SCRIPT 
      echo 192.168.56.11 master2.puppet.vm puppet >> /etc/hosts && rpm -Uvh https://yum.puppet.com/puppet6-release-el-8.noarch.rpm &&
      yum -y install puppet-agent && systemctl stop firewalld.service && systemctl disable firewalld.service
      SCRIPT
     }  
  ]

  servers.each do |machine|
    config.vm.define machine[:hostname] do |node|
        node.vm.box = machine[:box] # imposto sistema operativo
        node.vm.hostname = machine[:hostname] # imposto hostname
        #node.vm.network :private_network, ip: machine[:ip]
        #node.vm.network "forwarded_port", guest: 22, host: machine[:ssh_port], id: "ssh"
        node.vm.synced_folder machine[:shared_dir_host], machine[:shared_dir_guest] # folder condivisa da dentro a fuori vagrant
        node.vm.provision "file", source: "./copiedfile.txt", destination: "/home/vagrant/copiedfile.txt"
        #node.vm.network "forwarded_port", guest: 8140, host: 2223, protocol: "tcp"

        #imposto risorse HW della VM - parlando con VirtualBox
        node.vm.provider :virtualbox do |vb|
            vb.memory = 1024
            vb.cpus = 2
            vb.customize ['modifyvm', :id, '--natdnshostresolver1', 'on']            
        end

        #configuro comando da lanciare sulla macchina
        node.vm.provision "shell", inline: machine[:script]

        #imposto IP privato
        node.vm.network 'private_network', ip: machine[:private_address]
        ENV['LC_ALL']='en_US.UTF-8'
    end
end

    
  



#  config.vm.box = "generic/centos8"
#  config.vm.hostname = "master.puppet.vm"

#  config.vm.provider "virtualbox" do |v|
#    v.name = "master.puppet.vm"
#    v.memory = 2048
#    v.cpus = 2
#  config.vm.provision "shell", inline: <<-SHELL
#    sudo yum update
#  SHELL
#  end

end