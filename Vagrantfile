Vagrant.configure("2") do |config|


  #list of machines that will be created:
  
  servers=[
  
    
    # on the puppet master, execute updates and install puppetserver package. To make communication between machines functioning, a private network will be used
    # to make communication functioning disable and stop firewalld service
    # change the memory allocated for the puppet server, 2 GB are configured by default, in this case we restrict to 512 MB
    # start puppetserver service
    # enable puppetserver service
    # insert the IP of the server in the file hosts and associate it with 'puppet'
    {
      :hostname => "master2.puppet.vm",
	    :box => "generic/centos8",
      :private_address => '192.168.56.11', #private network
      :shared_dir_host => "./master_code",
      :shared_dir_guest => "/etc/puppetlabs/",
      :memory => 2048,
      :script => <<-SCRIPT 
      echo I am provisioning...
      date > /etc/vagrant_provisioned_at && sudo yum update -y && 
      rpm -Uvh https://yum.puppet.com/puppet6-release-el-8.noarch.rpm && yum install puppetserver git -y && sudo systemctl stop firewalld.service && systemctl disable firewalld.service &&
      sed -i 's/Xms2g/Xms512m/g' /etc/sysconfig/puppetserver && sed -i 's/Xmx2g/Xmx512m/g' /etc/sysconfig/puppetserver && systemctl start puppetserver &&
      systemctl enable puppetserver && echo 192.168.56.11 master2.puppet.vm puppet >> /etc/hosts
      SCRIPT
  
    },
	

    #generic node without Puppet
  	{
     :hostname => "rocky2.puppet.vm",
	   :box => "generic/rocky8",  
     :script => "",
     :private_address => '192.168.56.13', #private network
     :shared_dir_host => "./data",
     :shared_dir_guest => "/home/vagrant/data",
     :memory => 1024
    },

    # for a puppet node, download and install the puppet agent
    # download and install system updates
    # insert hostname/IP of master in /etc/hosts
    # to make communication functioning disable and stop firewalld service
    {
      :hostname => "clientc.puppet.vm",
      :box => "generic/centos8", 
      :private_address => '192.168.56.14', #private network
      :shared_dir_host => "./data",
      :shared_dir_guest => "/home/vagrant/data",
      :memory => 1024,
      :script => <<-SCRIPT 
      sudo yum update -y && 
      echo 192.168.56.11 master2.puppet.vm puppet >> /etc/hosts && rpm -Uvh https://yum.puppet.com/puppet6-release-el-8.noarch.rpm && 
      yum -y install puppet-agent && systemctl stop firewalld.service && systemctl disable firewalld.service 
      SCRIPT
     }  
  ]

  servers.each do |machine|
    config.vm.define machine[:hostname] do |node|
        node.vm.box = machine[:box] # set the operating system
        node.vm.hostname = machine[:hostname] # set the hostname
        node.vm.synced_folder machine[:shared_dir_host], machine[:shared_dir_guest] # shared folder for vagrant and host system
        node.vm.provision "file", source: "./copiedfile.txt", destination: "/home/vagrant/copiedfile.txt"

        # set HW resources of VM - talking with VirtualBox
        node.vm.provider :virtualbox do |vb|
            vb.memory = machine[:memory]
            vb.cpus = 2
            vb.customize ['modifyvm', :id, '--natdnshostresolver1', 'on']            
        end

        #configuro comando da lanciare sulla macchina
        node.vm.provision "shell", inline: machine[:script]

        #set private IP address
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