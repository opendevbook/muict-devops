## การใช้งาน คำสั่ง Ansible Ad-hoc
คำสั่งชนิด ad-hoc เป็นคำสั่งที่ ทำงานครั้งเดียว (one-time) สามารถ ทำงานได้โดยไม่ต้องมี playbook file

### การติดตั้ง ansible on centos 9 stream, redhat
```
sudo dnf install ansible
sudo dnf install sshpass
```

### เพิ่ม SSH Fingerprint ใน known_hosts
- หาก ต้องการ ให้ ansible ใช้ username password แต่เนื่องจากใน Config ของ ssh (/etc/ssh/sshd_config) มีการเปิดการใช้งาน SSH key สำหรับการ Authentication จึงจำเป็นต้องเพิ่ม SSH-fingerprints ใน known_hosts 

```
$ ip a
$ ssh vagrant@192.168.124.211
The authenticity of host '192.168.124.211 (192.168.124.211)' can't be established.
ED25519 key fingerprint is SHA256:O/Ndu7yOMoJbwvxpHBxn6UrTTgX8wrlFwV9QK0FgtDk.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '192.168.124.211' (ED25519) to the list of known hosts.
vagrant@192.168.124.211's password: 

This system is built by the Bento project by Chef Software
More information can be found at https://github.com/chef/bento

Use of this system is acceptance of the OS vendor EULA and License Agreements.
Last login: Sun Aug 11 14:30:21 2024 from 192.168.124.1

$ exit
```

- check know_hosts
```
$ cat ~/.ssh/known_hosts
192.168.124.211 ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIMPIb0hYL57yhPKaBuga7fZ05cQOmZJ1mGsknkRfUApy
192.168.124.211 ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQC6A2kqijwRgpApBwBIzBZuxiNo05a1z7vDuo73v412ojCiG38WE5NP1SASapwPphC/FXDb/gnRk1ejZwtZqT3/4zTKvf7YI2TyoDeaX0+pmVl9MJSTgYO0iEwDRgIUAOAy4vuGvVUwzU8+NqviLq++ELmXVqJUJWNUJ3BE+3HNLvGwR/n2KK8zwoQS5HqD6bBkoLff2Oglql1jQQJfaaNVaniLhCx32DhvpUHzWtQ1Kk4LsyULh8/Nh3qFB0np3qpXdSmlPhwnPT/CVuuDDeQMXsRnb9sonfhSCeyMCIQJ2Dj66yyEpAKzIdKq24hcK8/6oelrQO5Z+XSZZoKnmJIf6cYwavK+I1+NeqzfqB2UhZj+QW9ZbVHSsPOkixjpWl55sGQapKdcPKUbxCIY4LwrC7vaBExXG28PFW1ptusNLsnZCyVVAePkC3bg5J+50U56IXtblxga4JI6biJthTYFNBXIiN56omrQ22kweKB+fjxpOmc7xeazZemvtetNPjM=
192.168.124.211 ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBGqM/KcB6v83vkGaZjedBtYppyr5DERMM0ZIzhlRHfxGXCjbo0dI0/c7lyDBQM8yJ0rJpKxDEF7OK3nXmTZWxJ0=

```

ใน SSH protocol  `know_hosts`  คือเป็นคุณสมบัติดด้านความปลอดภัย ที่จะเก็บค่า fingerprint ของ remove server ที่เราจะ connected  fingerprints คือค่า ที่ได้จากการ hashed ข้อมูลของ sever ทำให้ค่าเฉพาะ (unique identifiers) เป็นการทำให้มั่นใจ ว่าเราทำการเชื่อมต่อไปยัง server นั้นๆ จริง

- **Options** หากไม่ต้องการให้ ansible ตรวจสอบ สามารถกำหนด ค่า defaults ใน /etc/ansible/ansible.cfg
(ไม่แนะนำ ใน production)
```
[defaults]
host_key_checking = False
```

- **Production option**
เพื่อให้ มีความปลอดภัยมาขั้น แนะนำให้ใช้ ssh keys แทน การใช้ password  ซึ่งจะมีการ Generate 

1 **Generate SSH key pair**
```
$ ssh-keygen -t rsa -b 2048
```

2 **copy public key ** ไปยัง server ปลายทาง
```
ssh-copy-id vagrant@192.168.124.211
```
> แทน ipserver ด้วย ip

3 **Update Host inventory**
```
[web]
server1 ansible_host=192.168.124.211 ansible_user=vagrant ansible_ssh_private_key_file=~/.ssh/id_rsa

```

4 **Run ansible command**
```
ansible -i hosts web -m ping
```

## สรุปคำสั่ง ad-hoc

ตั้งค่า  inventory  (/etc/ansible/hosts)
```
[web]
server1 ansible_host=192.168.124.153
```

ตัวอย่างคำสั่ง
```
$ ansible all  -m ping -e ansible_user=vagrant -e ansible_password=vagrant
```


เราไม่จำเป็นต้องระบุ `-e ansible_password=vagrant` แต่สามารถ ใช้ `-k` เพื่อให้รอรับคำสั่ง
```
$ ansible all -m ping -e ansible_user=vagrant -k
SSH password: 

```

เราสามารถที่จะระบุ ansible_user และ  ansible_password ต่อท้าย list ของ inventory และสามารถระบุให้ต่างกันได้ ไม่จำเป็นต้องเหมือนกัน
```
[web]
server1 ansible_host=192.168.124.153 ansible_user=vagrant ansible_password=vagrant
```
>หมายเหตุ  เปลี่ยน ip ให้เป็น ip ที่ต้องกับ hosts

1 **ทดสอบ การ เข้าถึง server ใน hosts**  
คำสั่งที่ใช้สำหรับการ check ว่า hosts ที่อยู่ใน inventory สามารถ เข้าถึงได้ไหม (reachable) ไม่ใช่แต่ ping แต่ต้องได้รับการ authentication ไม่ว่าจะเป็นการใช้ password หรือการใช้ ssh key ก็ตาม
```
ansible all -m ping

server1 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
```

2 **ตรวจสอบการใช้งาน การใช้ disk**
ตรวจสอบการใชงาน disk ของ ทุก Host
```
$ ansible all -m shell -a "df -h"

server1 | CHANGED | rc=0 >>
Filesystem      Size  Used Avail Use% Mounted on
/dev/vda2        15G  1.9G   14G  13% /
devtmpfs        4.0M     0  4.0M   0% /dev
tmpfs           227M  136K  227M   1% /dev/shm
tmpfs            91M  1.8M   89M   2% /run
tmpfs            46M  4.0K   46M   1% /run/user/1000

$ ansible all  -m shell  -a "ls -l"
server1 | CHANGED | rc=0 >>
total 4
-rw-r--r--. 1 vagrant vagrant 43 Aug 11 14:38 hosts
```

3  **ทำการสำเนา Copy file ไปยังทุก hosts**
```
ansible all -m copy -a "src=/path/to/local/file dest=/path/to/remote/destination"
```

5 **สั่ง คำสั่ง shell ในทุกๆ host**
ใช้ option -a
```
ansible all -a "ls -l"
```

5 **restart service ในทุกๆ host**
ยกตัวอย่างเช่น ต้องการ restart nginx ด้วย module service
```
ansible all -m service -a "name=nginx state=restarted"
```

6 **ติดตั้ง package** ด้วย module yum
ต้องการติดตั้ง package http
```
ansible all -m yum -a "name=httpd state=present" --become
```
> `--become` จะใช้เพื่อให้การติดตั้ง ด้วย สิทธิ sudo เนื่องจาก user vagrant มีสิทธิ sudo

```
server1 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": true,
    "msg": "",
    "rc": 0,
    "results": [
        "Installed: apr-util-openssl-1.6.3-16.fc40.x86_64",
        "Installed: httpd-tools-2.4.62-2.fc40.x86_64",
        "Installed: julietaula-montserrat-fonts-1:7.222-8.fc40.noarch",
        "Installed: httpd-2.4.62-2.fc40.x86_64",
        "Installed: httpd-core-2.4.62-2.fc40.x86_64",
        "Installed: apr-1.7.3-8.fc40.x86_64",
        "Installed: httpd-filesystem-2.4.62-2.fc40.noarch",
        "Installed: mailcap-2.1.54-5.fc40.noarch",
        "Installed: mod_http2-2.0.29-1.fc40.x86_64",
        "Installed: apr-util-1.6.3-16.fc40.x86_64",
        "Installed: mod_lua-2.4.62-2.fc40.x86_64",
        "Installed: fedora-logos-httpd-38.1.0-5.fc40.noarch",
        "Installed: apr-util-lmdb-1.6.3-16.fc40.x86_64"
    ]
}
```

**คำอธิบายเพิ่มเติม**
```
ansible all -m yum -a "name=httpd state=present" --become --ask-become-pass
```
> เราแทน `--ask-become-pass` ด้วย `-K`

```
$ ansible all -m yum -a "name=httpd state=present" --become -K
BECOME password: 
```

Restart service httpd
```
$ ansible all -m service -a "name=httpd state=started" --become
```

> State สามารถเลือกได้ จาก option ได้แก่ reloaded, restarted, started, stopped

```
$ ansible all -m service -a "name=httpd state=stopped" --become
```
