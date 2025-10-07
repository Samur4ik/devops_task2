# Указываем сервер для обхода блокировки в РФ
ENV['VAGRANT_SERVER_URL'] = 'https://vagrant.elab.pro'

# Используем образ Ubuntu 24.04
BoxName = "bento/ubuntu-24.04"
Provider = "virtualbox"

# Файл публичного SSH-ключа
SSH_PUB_KEY = "lab.pub"

# Определяем параметры ВМ
VirtMachine = Struct.new(:name, :ip, :cpus, :memory)

ansible = VirtMachine.new("ansible-host", "10.10.10.20", 2, 2048)
worker = VirtMachine.new("task-service", "10.10.10.30", 2, 2048)

vms = [ansible, worker]

Vagrant.configure("2") do |config|
  vms.each do |vm|
    config.vm.define vm.name do |cfg|
      cfg.vm.box = BoxName
      cfg.vm.hostname = "#{vm.name}.local"
      
      # Статический IP
      cfg.vm.network "private_network", ip: vm.ip
      
      # Проброс портов только для ansible-host
      if vm.name == "ansible-host"
        cfg.vm.network "forwarded_port", guest: 3001, host: 3001  # Backend
        cfg.vm.network "forwarded_port", guest: 3002, host: 3002  # Frontend
        cfg.vm.network "forwarded_port", guest: 30000, host: 30000 # Adminer
      end
      
      # Работа с SSH-ключом
      cfg.vm.provision "file", 
        source: SSH_PUB_KEY, 
        destination: "/tmp/ansible_key.pub"
        
      cfg.vm.provision "shell", inline: <<-SHELL
        mkdir -p /home/vagrant/.ssh
        chmod 700 /home/vagrant/.ssh
        cat /tmp/ansible_key.pub >> /home/vagrant/.ssh/authorized_keys
        chmod 600 /home/vagrant/.ssh/authorized_keys
        rm -f /tmp/ansible_key.pub
      SHELL

      # Настройки VirtualBox
      cfg.vm.provider "virtualbox" do |vb|
        vb.name = vm.name
        vb.memory = vm.memory
        vb.cpus = vm.cpus
      end
    end
  end
end