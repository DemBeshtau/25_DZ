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
4. Установка на сервер пакета ipa-server:
```shell
yum install -y ipa-server
```
5. Установка на клиентские хосты пакета ipa-client:
```shell
yum install -y ipa-client
```
6. Для настройки сервера LDAP необходимо запустить соответствующий скрипт ipa-server-install. Во время его работы в интерактивном режиме необходимо указать имя сервера, имя домена и пароль администратора LDAP сервера:
```shell
ipa-server-install
```
&ensp;&ensp;После завершения настройки и внесения соответствующих изменений в файл /etc/hosts хостовой машины, появится возможность доступа к серверу LDAP через веб-интерфейс:<br/>
![изображение](https://github.com/user-attachments/assets/ef0c4864-cfd1-48dd-af57-39bc28ca11de)

7. Добавление клиентских хостов в домен производится с помощью скрипта ipa-client-install, который необходимо запустить на каждом из них:
```shell
ipa-client-install --mkhomedir --domain=OTUS.LAN --server=ipa.otus.lan --no-ntp -p admin -w admin1234
```
8. Проверка наличия клиентских хостов в домене сервере:<br/>
![изображение](https://github.com/user-attachments/assets/31f85763-f9cb-4014-b835-84040dd38444)

9. Добавление нового пользователя в домен с аутентификацией по SSH-ключам:
```shell
echo "admin1234" | ipa user-add "user" --first="Otus" --last="Test" --shell="/bin/bash" --sshpubkey="ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQCFy2yLxfS8beM+q0kwPtA6WjLISvz/rdkkNYxGCsZTBxLcnkzV2/XQM0PzVCSYJGiYZNMO1FR9e0cPFmvsuOSDZ/tYOdOb+aHTtjF9Z4xA7faaV7chbPqptAgtjeO6zt+KJ5U9N9NwVREiz3VBkxMJ9Yv4mRDvkuNd+ZndNdllpnzpYxUnpNTTJST1Hqk4SxCDOAFVPf9fwOA9J1V/UgfKGp9/xCpbeCMyY835h93sw5e7KBgWDtz6msSKwn0CBV2l15JD5r+lWUHBEuIE4QQjkh6ABM9oi19oeL5QTr3OGgyr6EXQA3QGYTYMWOO+/6c5PVtPRBIg/uYDy81dOlKERmOdF9UMiEJYQxSsVk5qGx8bKH2kXjNUG3Zhwmz6b9/6G+IQx3rsqr/u8k7BGt67/1RdkZk8Sz7sL2AcDsAhnRTwPUU+s80clxQLdo/lDQ/S2WKwnxbUWWkZiF23YPXQD/qvgr9Ik0KtYgDUcBbic12R4HxX7gLMX45wpNkojYGk+PjZo6xCshrrRJnVDAur1R4oP7+m+wl06W325gfBMk6x8+3odNSjkyAyYqSbUNqcOMNYjn6rcVl7wrDWu8crdjUoVzzOvN8puG+tXMxfqPrmo2OQiP2944R4QyiDeTTDluRtvAO+EDVKXdz0CdKprHYt5zDUXg3rJLCowe3U8w== root@ipa.otus.lan" --password
```
10. Проверка наличия нового пользователя в домене:<br/>
![изображение](https://github.com/user-attachments/assets/b244b868-d0b2-462b-a5d6-0c6bfe36c1bd)

11. Настройка фаерволла на сервере и клиентах:
```shell
firewall-cmd --permanent --add-service=ntp
firewall-cmd --permanent --add-service=http
firewall-cmd --permanent --add-service=https
firewall-cmd --permanent --add-service=ldap
firewall-cmd --permanent --add-service=ldaps
firewall-cmd --permanent --add-service=kerberos
firewall-cmd --permanent --add-service=kpasswd
firewall-cmd --permanent --add-service=dns
```
&ensp;&ensp;Для автоматического конфигурирования инфраструктуры с помощью Ansible, подготовлен плейбук playbook.yml.





