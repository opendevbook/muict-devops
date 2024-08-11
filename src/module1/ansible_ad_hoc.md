## การใช้งาน คำสั่ง Ansible Ad-hoc
คำสั่งชนิด ad-hoc เป็นคำสั่งที่ ทำงานครั้งเดียว (one-time) สามารถ ทำงานได้โดยไม่ต้องมี playbook file

### สรุปคำสั่ง ad-hoc
ตัวอย่างคำสั่ง
```
$ ansible all  -m ping -e ansible_user=vagrant -e ansible_password=vagrant
```

ตัวอย่าง inventory  (/etc/ansible/hosts)
```
[web]
server1 ansible_host=192.168.124.153
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

```

3  **ทำการสำเนา Copy file ไปยังทุก hosts**
```
ansible all -m copy -a "src=/path/to/local/file dest=/path/to/remote/destination"
```

4 **restart service ในทุกๆ host**
ยกตัวอย่างเช่น ต้องการ restart nginx
```
ansible all -m service -a "name=nginx state=restarted"
```

5 ติดตั้ง package
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
