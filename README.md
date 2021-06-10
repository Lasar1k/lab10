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
Как выяснилось, нет. Дальнейшие действия носят чисто теоретический характер, т.к. на практике я их осуществить не могу.
```
$ vagrant port                           # просмотр порта
$ vagrant status                         
$ vagrant ssh                            # подключение к виртуалке через ssh

$ vagrant snapshot list                  # просмотр снимков виртуалки
$ vagrant snapshot push                  # добавление снимка виртуалки
$ vagrant snapshot list                  
$ vagrant halt                           # выключение виртуалки
$ vagrant snapshot pop                   # открытие снимка виртуалки
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
$ vagrant plugin list                         # просмотр плагинов
$ vagrant up --provider=vmware_esxi           # запуск виртуальной машины с указанием провайдера
```
