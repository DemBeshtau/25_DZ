# LDAP. Централизованная авторизация и аутентификация.
1. Установить FreeIPA;
2. Написать Ansible-playbook для конфигурации клиента;
3. Настроить аутентификацию по SSH-ключам;
4. Фаерволл должен быть включен на сервере и на клиенте.
### Исходные данные ###
&ensp;&ensp;ПК на Linux c 8 ГБ ОЗУ или виртуальная машина (ВМ) с включенной Nested Virtualization.<br/>
&ensp;&ensp;Предварительно установленное и настроенное ПО:<br/>
&ensp;&ensp;&ensp;Hashicorp Vagrant (https://www.vagrantup.com/downloads);<br/>
&ensp;&ensp;&ensp;Oracle VirtualBox (https://www.virtualbox.org/wiki/Linux_Downloads).<br/>
&ensp;&ensp;&ensp;Ansible (https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html).<br/>
&ensp;&ensp;Все действия проводились с использованием Vagrant 2.4.0, VirtualBox 7.0.18, Ansible 9.4.0 и образа CentOS Stream 9. <br/>
### Ход решения ###
1. Установка часового пояса на хостах:
```shell
timdatectl set-timezone Europe/Moscow
```
2. Остановка работы системы SELinux:
```shell
setenforce 0
```
3. В отсутвии настроек DNS-сервера, разрешение имён производится путём внесения изменений в файл /etc/hosts:
```shell
nano /etc/hosts

127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
127.0.1.1 ipa.otus.lan ipa
192.168.56.10 ipa.otus.lan ipa
```

