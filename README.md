# Vagrant стенд для NFS

 Данный стенд мной брался из домашнего задания № 2, после чего был отредактирован, а именно:<br>

## Редактирование Vagrantfile

MACHINES = {<br>
        :otuslinux => {<br>
            :box_name => "centos/7",<br>
        :ip_addr => '192.168.11.101',<br>
	:disks => {<br>
		:sata1 => {<br>
			:dfile => './sata1.vdi',<br>
			:size => 250,<br>
			:port => 1<br>
		},<br>
		:sata2 => {<br>
                        :dfile => './sata2.vdi',<br>
                        :size => 250, # Megabytes<br>
			:port => 2<br>
		},<br>
                :sata3 => {<br>
                        :dfile => './sata3.vdi',<br>
                        :size => 250,<br>
                        :port => 3<br>
                },<br>
                :sata4 => {<br>
                        :dfile => './sata4.vdi',<br>
                        :size => 250, # Megabytes<br>
                        :port => 4<br>
                },<br>
		:sata5 => {<br>
                        :dfile => './sata5.vdi', # Путь, по<b которому будет создан файл диска<br>
                        :size => 250, # Размер диска в мегабайтах
			:port => 5 # Номер порта на который будет зацеплен диск<br>
		}<br>
	}<br>


  },<br>
}<br>

### было заменено на: <br>

MACHINES = {<br>
  :server => {<br>
        :box_name => "centos/7",<br>
        :box_version => "1804.02",<br>
        :ip_addr => '192.168.11.101',<br>
  },<br>
  :client => {<br>
        :box_name => "centos/7",<br>
        :box_version => "1804.02",<br>
        :ip_addr => '192.168.11.102',<br>
  },<br>
} <br>


так же, чтобы избежать ошибки ""rsync" could not be found on your PATH. Make sure that rsync is properly installed on your system and available on the PATH.", следуя руководству по ссылке https://qna.habr.com/q/271364, необходимо установить плагин vagrant-vbguest:

    vagrant plugin install vagrant-vbguest

и дописать в Vagrant файл:

    config.vm.synced_folder ".", "/vagrant", type: "virtualbox"

## Настройка сервера:


### Я производил отключение Selinux
Все указанные далее действия я делал от лица su

    setenforce 0
    echo SELINUX=disabled > /etc/selinux/config

SELinux (SELinux) — это система принудительного контроля доступа, реализованная на уровне ядра. Впервые эта система появилась в четвертой версии CentOS, а в 5 и 6 версии реализация была существенно дополнена и улучшена. Эти улучшения позволили SELinux стать универсальной системой, способной эффективно решать массу актуальных задач. Стоит помнить, что классическая система прав Unix применяется первой, и управление перейдет к SELinux только в том случае, если эта первичная проверка будет успешно пройдена.

  1.1 Некоторые актуальные задачи.


  Для того, чтобы понять, в чем состоит практическая ценность SELinux, рассмотрим несколько примеров, когда стандартная система контроля доступа недостаточна. Если SELinux отключен, то вам доступна только классическая дискреционная система контроля доступа, которая включает в себя DAC (избирательное управление доступом) или ACL(списки контроля доступа). То есть речь идет о манипулировании правами на запись, чтение и исполнение на уровне пользователей и групп пользователей, чего в некоторых случаях может быть совершенно недостаточно. Например:

  — Администратор не может в полной мере контролировать действия пользователя. Например, пользователь вполне способен дать всем остальным пользователям права на чтение собственных конфиденциальных файлов, таких как ключи SSH.

  — Процессы могут изменять настройки безопасности. Например, файлы, содержащие в себе почту пользователя должны быть доступны для чтения только одному конкретному пользователю, но почтовый клиент вполне может изменить права доступа так, что эти файлы будут доступны для чтения всем.

  — Процессы наследуют права пользователя, который их запустил. Например, зараженная трояном версия браузера Firefox в состоянии читать SSH-ключи пользователя, хотя не имеет для того никаких оснований.

  В некоторых дистрибутивах SELinux имеется по умолчанию: CentOS, Fedora и Red Hat. В Debian и Ubuntu SeLinux включен в репозиторий. Установить его можно командой

    sudo apt-get install selinux

  затем перезапустить систему.

  Чтобы включить SELinux, необходимо ввести команду

    selinuxenabled 1

  Для выключения используется команда

    selinuxenabled 0

  Режимы работы

  SELinux работает в двух режимах.

  Permissive – допускается нарушение политики безопасности. Факты нарушения только регистрируются в системном журнале. В этом случае утилита используется в роли фиксатора нарушений.

  Enforced – режим, при котором нарушения политики безопасности блокируются – SELinux работает в полной мере.

  ### Затем производилась установка необходимого софта:<br>

    yum install rpcbind nfs-utils -y -q

rpcbind - это сервер, преобразующий номера программ RPC в универсальные адреса. Он должен работать, чтобы была возможность делать RPC вызовы (RPC calls) на сервер, на котором он будет запущен.

### Включаем серверы NFS

    systemctl enable rpcbind nfs-server
    systemctl start rpcbind nfs-server

### Создаем каталоги

    mkdir -p /var/nfs/upload
    chmod -R 777 /var/nfs/upload

где ключ -p позволяет создавать подкаталоги, в сосдаваемой директории. А правад доступа 777 регламентируют такие действия, как: Запись в папку или файл, чтение, а так же выполнение файла.

### Установка моего любимого редактора Nano

    yum install Nano

### Добавляем NFS каталог в /etc/exports
для этого отредактируем файл: <br>

    nano /etc/exports

и допишем туда строку <br>

    /var/nfs 192.168.11.102(rw,sync,no_root_squash,no_all_squash)

где: <br>
rw — разрешение на запись<br>
ro — только чтение<br>
sync — синхронный режим доступа. sync (async) — указывает, что сервер должен отвечать на запросы только после записи на диск изменений, выполненных этими запросами. Опция async указывает серверу не ждать записи информации на диск, что повышает производительность, но понижает надежность<br>
no_root_squash — по умолчанию пользователь root на клиентской машине не будет иметь доступа к разделяемой директории сервера. Этой опцией мы снимаем это ограничение.<br>
no_all_squash — включение пользовательской авторизации<br>
all_squash — все подключения будут выполнятся от анонимного пользователя.<br>

### Демон nfs-server автоматически перечитывает файл /etc/exports, но бывает, что надо вручную запустить перечитывание конфига

    exportfs -r

### Стартуем файрвол и разрешаем в нём NFS

    systemctl start firewalld.service
    firewall-cmd --permanent --zone=public --add-service=nfs
    firewall-cmd --permanent --zone=public --add-service=mountd
    firewall-cmd --permanent --zone=public --add-service=rpc-bind
    firewall-cmd --permanent --add-port=4001/udp --zone=public
    firewall-cmd --permanent --add-port=4001/tcp --zone=public
    firewall-cmd --permanent --add-port=2049/tcp --zone=public
    firewall-cmd --permanent --add-port=2049/udp --zone=public
    firewall-cmd --reload

### Не забывает включить обратно Selinux командой: <br>

    selinuxenabled 1

## Настройка Клиента:

### Установим необходимый софт

    yum install nfs-utils -q -y

### Стартуем нужные сервисы

    systemctl start rpcbind
    systemctl enable rpcbind

### Создаем и монтируем NFS директорию по UDP

    mkdir /mnt/nfs-share
    mount -t nfs 192.168.11.101:/var/nfs/ /mnt/nfs-share/ -o udp

### Создадим файл test.txt в директории /mnt/nfs-share/upload

    touch /mnt/nfs-share/upload/test.txt

### Проверим наличие файла

    ls /mnt/nfs-share/upload

Аналогично проверяем наличие файла на сервере в нашей директории.


### Добавим монтирование NFS директории в /etc/fstab

    echo "192.168.11.101:/var/nfs /mnt/nfs-share/ nfs defaults,udp 0 0" >> /etc/fstab


# При выполнение домашнего задания я пользовался следующей документацией: <br>

1. https://www.info-x.org/freebsd/nastroika/nastroika_i_ispolzovanie_nfs.html<br>
2. https://wired-mind.info/post/272 <br>
3. https://habr.com/ru/company/kingservers/blog/209644/<br>
4. https://defcon.ru/os-security/1264/<br>
5. https://blog.iteducenter.ua/sysadmin-basics/chto-vy-znaete-o-selinux-krome-togo-chto-ego-nado-otklyuchit/<br>
6. https://itdraft.ru/2019/12/09/ustanovka-i-nastrojka-nfs-servera-klienta-v-centos-7/ <br>
7. https://docs.google.com/document/d/1zFK4AP7O3se-_jQb9N_fPiMrvGIL0SkwpGvPg9EBHRk/edit# <br>
8. https://help.ubuntu.ru/wiki/nfs <br>
9. https://andreyex.ru/operacionnaya-sistema-linux/kak-nastroit-nfs-network-file-system-na-rhel-centos-fedora-i-debian-ubuntu/ <br>
