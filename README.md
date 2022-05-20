## Отчёт

1. Скачиваю и инициализирую vagrant на боксе bento/ubuntu-19.10 в новой папке vagrant
```sh
$ sudo apt install vagrant
$ mkdir Vagrant
$ cd Vagrant
$ vagrant init bento/ubuntu-19.10
```
2. Возникает ошибка при попытке запустить виртуальную машину (как с помощью vagrant up, так и с помощью vagrant up --provider virtualbox)
Проблема возникает с версией virtualbox (она устанавливается некорректно, без модуля kernel)
Попробовал переустановить, но не помогло

Решил, что возможно не хватает настроек в Vagrantfile, поэтому внёс в файл все нужные для создания машины настройки

```sh
$ cat > Vagrantfile <<EOF
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
Vagrant.configure("2") do |config|

  config.vagrant.plugins = ["vagrant-vbguest"]
   config.vm.box = "bento/ubuntu-19.10"
  config.vm.network "public_network"
  config.vm.synced_folder('shared', '/vagrant', type: 'rsync')

  config.vm.provider "virtualbox" do |vb|
    vb.gui = true
    vb.memory = "2048"
  end

  config.vm.provision "shell", inline: \$script, privileged: true

  config.ssh.extra_args = "-tt"

end
EOF
```

Однако это тоже не помогло, даже добавило ещё одну проблему - не установлен плагин vagrant-vbguest, попробовал его установить с помощью команды vagrant plugin install vagrant-vbguest

Но и тут не без проблем - оказывается плагины для вагранта недоступны теперь на территории РФ, поэтому пришлось воспользоваться VPN

Плагин установил, но проблема с запуском машины не решилась (даже с впн не запускает)
Странность ещё в том, что вручную с помощью программы можно создать машину на virtualbox, но через вагрант не получается

Ещё попробовал установить kernel вручную, но это так же не помогло

Пробовал также менять бокс на ubuntu/trusty64 (советы из интернета, но не помогли)

![Снимок экрана от 2022-05-20 23-04-40](https://user-images.githubusercontent.com/90759633/169603009-acb11714-dea5-47ca-8da6-719fbbd14efa.png)

Проблема оказалась в том, что у меня версия ОС Ubuntu 20.04, а у соответствующей версии virtualbox проблемы с совместимостью с этим ОС (разработчики видимо не договорились). Это всё вычитал в интернете в поисках решения проблемы

В теории можно было бы решить проблему, понизив версию Ubuntu, но как сделать это без сноса ОС я не нашёл, поэтому прибегнул к другому варианту

3. Создал с помощью Github Actions yml-файл (vagrant.yml в данном репозитории) и внёс туда все команды, которые я делал в консоли
```sh
name: vagrant

on: push

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: vagrant install
        run: |
          sudo apt install vagrant
          vagrant plugin install vagrant-vbguest
          sudo apt install virtualbox
      - name: vagrant init
        run: |
          vagrant init bento/ubuntu-19.10
          less Vagrantfile
          vagrant init -f -m bento/ubuntu-19.10
      - name: vagrantfile edit
        run: |
          mkdir shared
          cat > Vagrantfile <<EOF
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
          cat >> Vagrantfile <<EOF
          Vagrant.configure("2") do |config|
          config.vagrant.plugins = ["vagrant-vbguest"]
          EOF
          cat >> Vagrantfile <<EOF
            config.vm.box = "bento/ubuntu-19.10"
            config.vm.network "public_network"
            config.vm.synced_folder('shared', '/vagrant', type: 'rsync')
          
            config.vm.provider "virtualbox" do |vb|
              vb.gui = true
              vb.memory = "2048"
            end
            config.vm.provision "shell", inline: \$script, privileged: true
            config.ssh.extra_args = "-tt"
          end
          EOF
      - name: vagrant validate
        run: |
          vagrant validate
          vagrant status
      - name: vagrant up
        run: |
          VAGRANT_DETECTED_OS=cygwin
          vagrant up --provider virtualbox
          vagrant port
          vagrant ssh
      - name: vagrant destroy
        run: |
          vagrant destroy --force
```

4. По итогу прогресс: вагрант смог сертифицировать вагрант файл ($ vagrant validate), однако сама машина хоть и грузится, но не запускается

![Снимок экрана от 2022-05-20 23-19-42](https://user-images.githubusercontent.com/90759633/169604745-1133ab89-9339-4e18-9768-267dfa5fa285.png)
![Снимок экрана от 2022-05-20 23-20-14](https://user-images.githubusercontent.com/90759633/169604807-28f3c459-0154-4265-93d7-34abd5b58505.png)

Вагрант требует TTY. Я начал искать решение этой проблемы в интернете, по итогу нашёл форум, где всем помогало поставить cygwin, однако мне к сожалению не помогло

```sh
VAGRANT_DETECTED_OS=cygwin
```

Я честно пытался решить около недели эту проблему, но по итогу ни к чему не пришёл, хотя всё должно работать


## Laboratory work X

Данная лабораторная работа посвещена изучению процесса создания и конфигурирования виртуальной среды разработки с использованием **Vagrant**

```shб
$ open https://www.vagrantup.com/intro/index.html
```

## Tasks

- [ ] 1. Ознакомиться со ссылками учебного материала
- [ ] 2. Выполнить инструкцию учебного материала
- [ ] 3. Составить отчет и отправить ссылку личным сообщением в **Slack**

## Tutorial

```sh
$ export GITHUB_USERNAME=<имя_пользователя>
$ export PACKAGE_MANAGER=<пакетный_менеджер>
```

```sh
$ cd ${GITHUB_USERNAME}/workspace
$ ${PACKAGE_MANAGER} install vagrant
```

```sh
$ vagrant version
$ vagrant init bento/ubuntu-19.10
$ less Vagrantfile
$ vagrant init -f -m bento/ubuntu-19.10
```

```sh
$ mkdir shared
```

```sh
$ cat > Vagrantfile <<EOF
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

```sh
$ cat >> Vagrantfile <<EOF

Vagrant.configure("2") do |config|

  config.vagrant.plugins = ["vagrant-vbguest"]
EOF
```

```sh
$ cat >> Vagrantfile <<EOF

  config.vm.box = "bento/ubuntu-19.10"
  config.vm.network "public_network"
  config.vm.synced_folder('shared', '/vagrant', type: 'rsync')

  config.vm.provider "virtualbox" do |vb|
    vb.gui = true
    vb.memory = "2048"
  end

  config.vm.provision "shell", inline: \$script, privileged: true

  config.ssh.extra_args = "-tt"

end
EOF
```

```sh
$ vagrant validate

$ vagrant status
$ vagrant up # --provider virtualbox
$ vagrant port
$ vagrant status
$ vagrant ssh

$ vagrant snapshot list
$ vagrant snapshot push
$ vagrant snapshot list
$ vagrant halt
$ vagrant snapshot pop
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
$ vagrant plugin install vagrant-vmware-esxi
$ vagrant plugin list
$ vagrant up --provider=vmware_esxi
```

## Report

```sh
$ cd ~/workspace/
$ export LAB_NUMBER=10
$ git clone https://github.com/tp-labs/lab${LAB_NUMBER}.git tasks/lab${LAB_NUMBER}
$ mkdir reports/lab${LAB_NUMBER}
$ cp tasks/lab${LAB_NUMBER}/README.md reports/lab${LAB_NUMBER}/REPORT.md
$ cd reports/lab${LAB_NUMBER}
$ edit REPORT.md
$ gist REPORT.md
```

## Links

- [VirualBox](https://www.virtualbox.org/)
- [Vagrant providers](https://github.com/hashicorp/vagrant/wiki/Available-Vagrant-Plugins#providers)
- [Vagrant vbguest plugin](https://github.com/dotless-de/vagrant-vbguest)
- [Vagrant disksize plugin](https://github.com/sprotheroe/vagrant-disksize)
- [Vagrant vmware esxi plugin](https://github.com/josenk/vagrant-vmware-esxi)

```
Copyright (c) 2015-2021 The ISC Authors
```
