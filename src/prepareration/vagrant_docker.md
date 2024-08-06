# vagrant workshop: Docker host

## วัตถุประสงค์
เรียนรู้วิธีการสร้าง Docker host สำหรับการเรียนรู้ Container

## ขั้นตอนการสร้าง Single Node LAMP Stack ด้วย Vagrant

### 1. การตั้งค่าโปรเจค
1. เปิดเทอร์มินัลหรือ PowerShell
2. สร้างไดเรกทอรีใหม่สำหรับโปรเจค
    ```sh
    mkdir vagrant-docker
    cd vagrant-docker
    ```
3. รันคำสั่ง `vagrant init` เพื่อสร้างไฟล์ `Vagrantfile`
    ```sh
    vagrant init generic/ubuntu2304
    ```

### 2. การกำหนดค่า Vagrantfile
เปิดไฟล์ `Vagrantfile` และแก้ไขดังนี้:
```ruby
# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.

$script = <<-SCRIPT

  # ติดตั้ง packages
  sudo apt install apt-transport-https ca-certificates curl software-properties-common -y
  
  # ติดตั้ง Docker official GPG key
  curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

  # ติดตั้ง Repository
  sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" -y
  sudo apt-cache policy docker-ce 

  # ติดตั้ง Docker
  sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y 
  sudo usermod -aG docker vagrant 

  # เริ่มต้น Docker service
  sudo systemctl enable --now docker
 
SCRIPT
Vagrant.configure("2") do |config|
  config.vm.box = "generic/ubuntu2304"
  config.vm.network "private_network", ip: "192.168.33.10"
  config.vm.synced_folder ".", "/vagrant", type: 'rsync'

  
  config.vm.provider "virtualbox" do |vb|
    vb.cpus = 2
    vb.memory = 2048
  end

  config.vm.provision "shell" , inline: $script
end


```

### 3. การเริ่มต้น VM
รันคำสั่ง vagrant up เพื่อสร้างและเริ่มต้นเครื่องเสมือน

```
vagrant up --provider=virtualbox --provision
```

### 4. รันคำสั่ง vagrant ssh เพื่อเชื่อมต่อกับเครื่องเสมือนผ่าน SSH

```
vagrant ssh
```

### 5. ทดสอบคำสั่ง ``docker ps``

```
vagrant@ubuntu2304:~$ docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```

### 6. ทดสอบเลือก pull image จาก docker hub [https://hub.docker.com/search?image_filter=official](https://hub.docker.com/search?image_filter=official)

```
vagrant@ubuntu2304:~$ docker pull python
Using default tag: latest
latest: Pulling from library/python
ca4e5d672725: Pull complete
30b93c12a9c9: Pull complete
10d643a5fa82: Pull complete
d6dc1019d793: Pull complete
5afcd2745721: Pull complete
b5dbe4a95907: Pull complete
26b10780b7b0: Pull complete
d2f126f49360: Pull complete
Digest: sha256:e8be0ea148390d08bc077840cf87ac6a538d80b0ea1e8752b3e3982987cd0a53
Status: Downloaded newer image for python:latest
docker.io/library/python:latest
```

