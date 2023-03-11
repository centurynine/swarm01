# Ref
- https://github.com/docker/awesome-compose/tree/master/django

# Wakatime
- https://wakatime.com/@spcn05/projects/qxrsnbwxam

# Revert proxy
- https://spcn05-php.xops.ipv9.me/

# Setup VM

1. สร้าง VM
    - Ubuntu 22.04
    - 2 core CPU
    - 2 GB of ram
    - 32 GB HDD

2. Clone VM to template

3. ทำการสร้าง VM จาก template มาทั้งหมด 3 node
    - manager
    - worker1
    - worker2

4. ทำการเปลี่ยนค่าชื่อ hostname ทุกโหนด
```
hostnamectl set-hostname "name"
```
5. รีเซ็ทค่า machine-id ทุกโหนดเพื่อไม่ให้ ip ชนกัน
```
cp /dev/null /etc/machine-id
rm /var/lib/dbus/machine-id
ln -s /etc/machine-id /var/lib/dbus/machine-id
```

# Docker swarm
1. ทำการกำหนดเครื่อง manager
```
docker swarm init
```
    - โดยจะได้คำสั่งและคีย์มาเพื่อนำไปสร้าง worker

2. นำคำสั่งที่ได้มาไปรันบนเครื่อง worker

3. ทำการติดตั้ง Portainer ทั้ง 3 node เพื่อใช้เป็นตัว monitoring และทำให้ง่ายต่อการจัดการ
```
curl -L https://downloads.portainer.io/ce2-17/portainer-agent-stack.yml -o portainer-agent-stack.yml
docker stack deploy -c portainer-agent-stack.yml portainer
```

# Traefik
1. ทำการแก้ไขไฟล์ host บนเครื่องคอมพิวเตอร์
    - Windows C:\Windows\System32\drivers\etc\hosts
    - Linux/Mac /etc/hosts
    โดยเพิ่ม
    ```
    172.31.1.xxx portainer.cpedemo.local
    172.31.1.xxx traefik.cpedemo.local
    172.31.1.xxx swarmpit.cpedemo.local
    ```
2. ทำการติดตั้ง traefik เพื่อใช้ในการ dashboard ดู http routers
    ```
    docker network create --driver=overlay traefik-public

    export NODE_ID=$(docker info -f '{{.Swarm.NodeID}}') 
    echo $NODE_ID

    docker node update --label-add traefik-public.traefik-public-certificates=true $NODE_ID

    export EMAIL=<email>
    export DOMAIN=<domain>
    export USERNAME=admin
    export PASSWORD=<password>
    export HASHED_PASSWORD=$(openssl passwd -apr1 $PASSWORD)
    echo $HASHED_PASSWORD

    docker stack deploy -c traefik-host.yml traefik
    ```

3. ตรวจสอบการใช้งานโดยเข้าไปที่ https://traefik.cpedemo.local/