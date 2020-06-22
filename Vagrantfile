# -*- mode: ruby -*-
# vim: set ft=ruby :

MACHINES = {
  :server => {
        :box_name => "centos/7",
        :box_version => "1804.02",
        :ip_addr => '192.168.11.101',
  },
  :client => {
        :box_name => "centos/7",
        :box_version => "1804.02",
        :ip_addr => '192.168.11.102',
  },
}

Vagrant.configure("2") do |config|
	config.vm.synced_folder ".", "/vagrant", type: "virtualbox"
	config.vm.box_version = "1804.02"
		MACHINES.each do |boxname, boxconfig|
			config.vm.define boxname do |box|
			box.vm.box = boxconfig[:box_name]
			box.vm.host_name = boxname.to_s
			#box.vm.network "forwarded_port", guest: 3260, host: 3260+offset
			box.vm.network "private_network", ip: boxconfig[:ip_addr]
			box.vm.provider :virtualbox do |vb|
				vb.customize ["modifyvm", :id, "--memory", "1024"]
               		end
			box.vm.provision "shell", inline: <<-SHELL
				mkdir -p ~root/.ssh
				cp ~vagrant/.ssh/auth* ~root/.ssh
			SHELL
			end
		end
	config.vm.define "server" do |server|
        	server.vm.provision "shell", inline: <<-SHELL
		echo "Устанавливаем гостевой плагин"
			vagrant plugin install vagrant-vbguest
		echo "Отключаем SELINUX"
            		setenforce 0
            	echo SELINUX=disabled > /etc/selinux/config
		echo "Установим необходимый софт"
        		yum install rpcbind nfs-utils -y -q > /dev/null 2>&1
        	echo "Включаем серверы NFS"
        		systemctl enable rpcbind nfs-server > /dev/null 2>&1
        		systemctl start rpcbind nfs-server
		echo "Создаем каталоги"
        		mkdir -p /var/nfs/upload
        		chmod -R 777 /var/nfs/upload
		echo "Установка текстового редактора"
			yum install Nano
		echo "Добавляем NFS каталог в /etc/exports"
            		echo "/var/nfs 192.168.11.0/24(rw,sync,no_root_squash,no_all_squash)" >> /etc/exports
            	echo "Вручную запускаем перечитывание конфиг файла"
			exportfs -r
		echo "Стартуем файрволл и разрешаем в нём NFS"
            		systemctl start firewalld.service
            		firewall-cmd --permanent --zone=public --add-service=nfs > /dev/null 2>&1
            		firewall-cmd --permanent --zone=public --add-service=mountd > /dev/null 2>&1
            		firewall-cmd --permanent --zone=public --add-service=rpc-bind > /dev/null 2>&1
            		firewall-cmd --permanent --add-port=4001/udp --zone=public > /dev/null 2>&1
            		firewall-cmd --permanent --add-port=4001/tcp --zone=public > /dev/null 2>&1
            		firewall-cmd --permanent --add-port=2049/tcp --zone=public > /dev/null 2>&1
            		firewall-cmd --permanent --add-port=2049/udp --zone=public > /dev/null 2>&1
            		firewall-cmd --reload > /dev/null 2>&1
		echo "не забывает включить обратно Selinux командой:"
			selinuxenabled 1
		echo "Включаем поддержку вывода Escape последовательностей"
			echo -e "Далее настройка клиента.\n"
        SHELL
	end
       config.vm.define "client" do |client|
       		client.vm.provision "shell", inline: <<-SHELL
		echo "Установим необходимый софт"
          		yum install nfs-utils -q -y 
          	echo "Стартуем нужные сервисы"
          		systemctl start rpcbind
          		systemctl enable rpcbind 
          	echo "Создаем и монтируем NFS директорию по UDP"
          		mkdir /mnt/nfs-share
          		mount -t nfs 192.168.11.101:/var/nfs/ /mnt/nfs-share/ -o udp
		echo "Создадим файл test.txt в директории /mnt/nfs-share/upload"
          		touch /mnt/nfs-share/upload/test.txt
          	echo "вывод комманды ls /mnt/nfs-share/upload: `ls /mnt/nfs-share/upload`"
          	echo "Добавим монтирование NFS директории в /etc/fstab"
          		echo "192.168.11.101:/var/nfs /mnt/nfs-share/ nfs defaults,udp 0 0" >> /etc/fstab

       SHELL
       end
  end












