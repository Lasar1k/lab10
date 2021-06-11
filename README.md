## Laboratory work X

Данная лабораторная работа посвещена изучению процесса создания и конфигурирования виртуальной среды разработки с использованием **Vagrant**

```sh
$ open https://www.vagrantup.com/intro/index.html
```
Vagrant-это инструмент для создания и управления средами виртуальных машин в рамках одного рабочего процесса. Благодаря простому в использовании рабочему процессу и сосредоточенности на автоматизации, Vagrant сокращает время настройки среды разработки, увеличивает четность производства и делает оправдание "работы на моей машине" пережитком прошлого.
## Tasks

- [ ] 1. Ознакомиться со ссылками учебного материала
- [ ] 2. Выполнить инструкцию учебного материала
- [ ] 3. Составить отчет и отправить ссылку личным сообщением в **Slack**

## Tutorial
PACKAGE_MANAGER у нас это apt
```sh
$ export GITHUB_USERNAME=<имя_пользователя>
$ export PACKAGE_MANAGER=<пакетный_менеджер>
```
При попытке ввести вторую команду терминал выдавал ошибку, т.к. не было рут-прав. Поэтому пишем sudo ${PACKAGE_MANAGER} install vagrant
```sh
$ cd ${GITHUB_USERNAME}/workspace
$ ${PACKAGE_MANAGER} install vagrant
```

```sh
$ vagrant version

Installed Version: 2.2.6

Vagrant was unable to check for the latest version of Vagrant.
Please check manually at https://www.vagrantup.com

$ vagrant init bento/ubuntu-19.10 ## инициализируем виртуалку на ubuntu

A `Vagrantfile` has been placed in this directory. You are now
ready to `vagrant up` your first virtual environment! Please read
the comments in the Vagrantfile as well as documentation on
`vagrantup.com` for more information on using Vagrant.

$ less Vagrantfile ## появится файл настроек виртуалки
$ vagrant init -f -m bento/ubuntu-19.10 ## Создаем новый Vagrantfile, перезаписываем изначальный (-f), находящийся в текущем пути. И информация в нем будет в минимальном объеме (-m)

A `Vagrantfile` has been placed in this directory. You are now
ready to `vagrant up` your first virtual environment! Please read
the comments in the Vagrantfile as well as documentation on
`vagrantup.com` for more information on using Vagrant.

```

```sh
$ mkdir shared
```

```sh
$ cat > Vagrantfile <<EOF ## добавляем настройки виртуалки
\$script = <<-SCRIPT
sudo apt install docker.io -y
sudo docker pull fastide/ubuntu:19.04
sudo docker create -ti --name fastide fastide/ubuntu:19.04 bash
sudo docker cp fastide:/home/developer /home/
sudo useradd developer
sudo usermod -aG sudo developer
echo "developer:developer" | sudo chpasswd
sudo chown -R developer /home/developer
SCRIPT
EOF
```
vagrant-vbguest - это плагин, который автоматически обновляет гостевые дополнения VirtualBox. Продолжаем настраивать виртуалку
```sh
$ cat >> Vagrantfile <<EOF

Vagrant.configure("2") do |config|

  config.vagrant.plugins = ["vagrant-vbguest"]
EOF
```
Продолжение настраивания 
```sh
$ cat >> Vagrantfile <<EOF

  config.vm.box = "bento/ubuntu-19.10"                            # Указываем версию виртуальной машины: ubuntu-19.10
  config.vm.network "public_network"                              # Указываем настройки сети: public_network
  config.vm.synced_folder('shared', '/vagrant', type: 'rsync')    # Указываем связующие директории: 'shared', '/vagrant', type: 'rsync'

  config.vm.provider "virtualbox" do |vb|                         # Указываем тип виртуальной машины: virtualbox
    vb.gui = true                                                 # Указываем, что используется графический интерфейс: vb.gui = true 
    vb.memory = "2048"                                            # Указываем, сколько выделяем оперативной памяти под виртуальную машину: 2048МБ 
  end

  config.vm.provision "shell", inline: \$script, privileged: true # config.vm.provision "shell" - задает встроенную команду оболочки для выполнения на удаленном компьютере

  config.ssh.extra_args = "-tt"                                   # config.ssh.extra_args - значение настроек передается непосредственно в исполняемый файл ssh#

end
EOF
```
На этом этапе обнаружилось, что нужно дополнительно установить плагин командой
```sh
$ vagrant plugin install vagrant-vbguest
```
```sh
$ vagrant validate                       # Проверка корректности файла Vagrantfile

$ vagrant status                         # Проверка статуса
$ vagrant up                             # запуск виртуалки
```
Здесь терминал опять сломался, т.к. у меня не было библиотеки libvirt. В ходе моих изысканий на оф.сайте ubuntu в документации я нашёл порядок действий, необходимых для 
установки этой библиотеки. Сначала нужно проверить, поддерживает ли вообще мой процессор виртуализацию
```
$ kvm-ok
INFO: Your CPU does not support KVM extensions
KVM acceleration can NOT be used

```
Как выяснилось, нет. Проблему я смог решить, переустановив заново vagrant с плагином и virtualbox
```
$ sudo vagrant up --provider virtualbox
Bringing machine 'default' up with 'virtualbox' provider...
	==> default: Box 'bento/ubuntu-19.10' could not be found. Attempting to find and install...
	    default: Box Provider: virtualbox
        ...

$ vagrant port                           # просмотр порта
  22 (guest) => 2222 (host)
$ vagrant status     
Current machine states:
	
	default                   running (virtualbox)
$ vagrant ssh                            # подключение к виртуалке через ssh
  System information as of Fri 11 Jun 2021 09:01:51 PM UTC
	
	   System load:  0.09              Processes:           110
	  Usage of /:   11.4% of 7.79GB   Users logged in:     0
	  Memory usage: 14%                IP address for eth0: 10.0.2.15
	  Swap usage:   0%
      ...
      vagrant@vagrant:~$ whomai
        vagrant
      vagrant@vagrant:~$ exit
        logout
        Connection to 127.0.0.1 closed.
        

$ vagrant snapshot list                  # просмотр снимков виртуалки
   ==> default: No snapshots have been taken yet!
$ vagrant snapshot push                  # добавление снимка виртуалки
==> default: Snapshotting the machine as 'push_1620758364_4319'...
	==> default: Snapshot saved! You can restore the snapshot at any time by
	==> default: using `vagrant snapshot restore`. You can delete it using
	==> default: `vagrant snapshot delete`.
$ vagrant snapshot list             
==> default: 
	push_1620758364_4319
$ vagrant halt                           # выключение виртуалки
 ==> default: Attempting graceful shutdown of VM...
$ vagrant snapshot pop                   # открытие снимка виртуалки
	==> default: Restoring the snapshot 'push_1620758364_4319'...
	==> default: Deleting the snapshot 'push_1620758364_4319'...
	==> default: Snapshot deleted!
        ...
```

```ruby
  config.vm.provider :vmware_esxi do |esxi|

    esxi.esxi_hostname = '<exsi_hostname>'
    esxi.esxi_username = 'root'
    esxi.esxi_password = 'prompt:'

    esxi.esxi_hostport = 22

    esxi.guest_name = '${GITHUB_USERNAME}'

    esxi.guest_username = 'vagrant'
    esxi.guest_memsize = '2048'
    esxi.guest_numvcpus = '2'
    esxi.guest_disk_type = 'thin'
  end
```

```sh
$ vagrant plugin install vagrant-vmware-esxi  # установка плагина vmware
 Installing the 'vagrant-vmware-esxi' plugin. This can take a few minutes...
    ...
    Installed the plugin 'vagrant-vmware-esxi (2.5.2)'!
$ vagrant plugin list                         # просмотр плагинов
  vagrant-vmware-esxi (2.5.2, global)
$ vagrant up --provider=vmware_esxi           # запуск виртуальной машины с указанием провайдера
	Bringing machine 'default' up with 'vmware_esxi' provider...
    ...
```
