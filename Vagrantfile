# Script de instalação do MySQL e criacao do usuario phpuser
$script_mysql = <<-SCRIPT
    apt-get update && \
    apt-get install -y mysql-server-5.7 && \
    mysql -e "create user 'phpuser'@'%' identified by 'pass';"
SCRIPT

# -----------------------------------------------------------------

Vagrant.configure("2") do |config|
  # Definição do Box a ser utilizado - Ubuntu 18.04 LTS
  config.vm.box = "ubuntu/bionic64"
  
  # ----------------------------------------------------------------
  
  #Configurações da VM MySQL Server
  config.vm.define "mysqlsrv" do |mysqlsrv|
    
  mysqlsrv.vm.network "forwarded_port", guest: 80, host: 8089

  #Configuração de Processamento, Memória e Hostname do Servidor
    mysqlsrv.vm.provider "virtualbox" do |mysqlsrv|
      mysqlsrv.memory = 768
      mysqlsrv.cpus = 2
      mysqlsrv.name = "srv-mysql"
    end
    
    #Definição do IP e tipo de rede
    mysqlsrv.vm.network "public_network", ip: "172.29.0.99"
  
    #Adição da chave pública na VM
    mysqlsrv.vm.provision "shell", 
      inline: "cat /configs/id_bionic.pub >> .ssh/authorized_keys"

    #Execução do script de instalacao e configuracao do MySQL  
    mysqlsrv.vm.provision "shell", 
      inline: $script_mysql
    
    #Configurando permissao de acesso remoto ao MySQL
    mysqlsrv.vm.provision "shell", 
      inline: "cat /configs/mysqld.cnf > /etc/mysql/mysql.conf.d/mysqld.cnf"

    #Reiniciando o MySQL após instalar
    mysqlsrv.vm.provision "shell", 
      inline: "service mysql restart"

    # Ativando compartilhamento da pasta /configs entre host fisico e VM
    # e desativando o compartilhamento padrão
    mysqlsrv.vm.synced_folder "./configs", "/configs"
    mysqlsrv.vm.synced_folder ".", "/vagrant", disabled: true
  end

  # ------------------------------------------------------------------------------

  #Configurações da VM PHPWeb

  config.vm.define "phpweb" do |phpweb|
    
    phpweb.vm.network "forwarded_port", guest: 8888, host: 8888

    #Definição do IP e tipo de rede
    phpweb.vm.network "public_network", ip: "172.29.0.98"

    #Configuração de Processamento, Memória e Hostname do Servidor
    phpweb.vm.provider "virtualbox" do |phpweb|
      phpweb.memory = 768
      phpweb.cpus = 2
      phpweb.name = "srv-phpweb"
    end

    # Instalação do Puppet após o provisionamento da VM
    phpweb.vm.provision "shell", 
      inline: "apt-get update && apt-get install -y puppet"

    # Cópia do Arquivo Manifest do Puppet (phpweb.pp) para a pasta correta 
    # após a instalação do Puppet
    phpweb.vm.provision "puppet" do |puppet|
      puppet.manifests_path = "./configs/manifest"
      puppet.manifest_file = "phpweb.pp"
    end

    # Ativando compartilhamento da pasta /configs entre host fisico e VM
    # e desativando o compartilhamento padrão
    phpweb.vm.synced_folder "./configs", "/configs"
    phpweb.vm.synced_folder ".", "/vagrant", disabled: true

  end

# -------------------------------------------------------------------------

#Configurações da VM Ansible

  config.vm.define "ansible" do |ansible|
    ansible.vm.network "public_network", ip: "172.29.0.96"

    #Configuração de Processamento, Memória e Hostname do Servidor
    ansible.vm.provider "virtualbox" do |ansible|
      ansible.memory = 768
      ansible.cpus = 2
      ansible.name = "srv-ansible"
    end

    #Adição da chave privada na VM, para que outras VMs consigam acessar o playbook.yml
    ansible.vm.provision "shell", 
      inline: "cp /vagrant/configs/id_bionic /home/vagrant && \
               chmod 600 /home/vagrant/id_bionic && \
               chmod vagrant:vagrant /home/vagrant/id_bionic"

    #Instalação do Ansible na VM
    ansible.vm.provision "shell",
            inline: "apt-get update && \
                     apt-get install -y software-properties-common && \
                     apt-add-repository --yes --update ppa:ansible/ansible && \
                     apt-get install -y ansible"

    # Ativando compartilhamento da pasta /configs entre host fisico e VM
    # e desativando o compartilhamento padrão
    ansible.vm.synced_folder "./configs", "/configs"
    ansible.vm.synced_folder ".", "/vagrant", disabled: true
    end
end

