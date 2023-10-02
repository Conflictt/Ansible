# `Перші кроки з Ansible`

Підготувати стенд на Vagrant як мінімум з одним сервером

## `Завдання`

На цьому сервері використовуючи Ansible потрібно розгорнути nginx з наступними умовами:

* Необхідно використовувати модуль yum/apt
* Конфігураційні файли повинні бути взяті із шаблону jinja2 зі змінними
* Після встановлення nginx повинен бути в режимі enabled в systemd
* Повинен бути використаний notify для старту nginx після встановлення
* Сайт повинен слухатись на нестандартному порті - 8080 (для цього використовувати змінні в Ansible)

### `Вирішення`

 Зберегти nginx.conf.j2, nginx.yml, Vagrantfile в один каталог

<details><summary>Vagrantfile CentOS 7</summary><blockquote>

```ruby
# -*- mode: ruby -*-
# vim: set ft=ruby :

MACHINES = {
  :nginx => {
        :box_name => "centos/7",
        :ip_addr => '192.168.11.150'
  }
}

Vagrant.configure("2") do |config|

  MACHINES.each do |boxname, boxconfig|

      config.vm.define boxname do |box|
        box.vm.box = boxconfig[:box_name]
        box.vm.host_name = boxname.to_s
        box.vm.network "private_network", ip: boxconfig[:ip_addr]
        box.vm.provider :virtualbox do |vb|
          vb.customize ["modifyvm", :id, "--memory", "200"]
        end
        box.vm.provision "shell", inline: <<-SHELL
          mkdir -p ~root/.ssh; cp ~vagrant/.ssh/auth* ~root/.ssh
          sed -i '65s/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config
          systemctl restart sshd
        SHELL
        box.vm.provision :ansible do |ansible|
        ansible.limit = "all"
        ansible.playbook = "nginx.yml"
        end
      end
  end
end
```

</blockquote></details>

<details><summary>Vagrantfile Debian 11</summary><blockquote>

```ruby
# -*- mode: ruby -*-
# vim: set ft=ruby :

ENV["LC_ALL"] = "en_US.UTF-8"

Vagrant.configure("2") do |config|

  config.vm.box = "Conflict/Debian11"
  config.vm.box_check_update = false
  
  config.vm.define "03-Ansible" do |ansible|
    ansible.vm.hostname = "03-Ansible"
    ansible.vm.network "public_network", ip: "192.168.88.238"

    ansible.vm.provider "virtualbox" do |vb|
      vb.check_guest_additions = false
      vb.name = "03-Ansible" 
      vb.gui = false
      vb.memory = 1024
      vb.cpus = 1
    end
    ansible.vm.provision :ansible do |ansible|
      ansible.limit = "all"
      ansible.playbook = "nginx.yml"
    end
  end
end
```

</blockquote></details>

Підняти віртуалку `vagrant up`

Дочекатись виконання процесу конфігурації, після чого можна перевірити доступність сайту командою `curl http://192.168.88.238:8080`

У файлі /usr/share/nginx/html/index.html можна змінити стартову сторінку nginx на наступний:

```html
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN">
<html>
<head>
  <title>Welcome to CentOS</title>
  <style rel="stylesheet" type="text/css">

html {
 background-image:url(img/html-background.png);
 background-color: white;
 font-family: "DejaVu Sans", "Liberation Sans", sans-serif;
 font-size: 0.85em;
 line-height: 1.25em;
 margin: 0 4% 0 4%;
}

body {
 border: 10px solid #fff;
 margin:0;
 padding:0;
 background: #fff;
```
