# MMDVM by G4KLX

![logo](https://i1.wp.com/www.gb7dd.co.uk/wp-content/uploads/2016/02/MMDVM-logo.jpg?w=900)

# Install Docker
```console
$ curl -fsSL get.docker.com -o get-docker.sh
$ sudo sh get-docker.sh
```

# Install docker-compose
* (Debian)
```console
$ sudo curl -L https://github.com/docker/compose/releases/download/1.21.0/docker-compose-$(uname -s)-$(uname -m) -o /usr/local/bin/docker-compose
$ chmod +x /usr/local/bin/docker-compose
```

* (Raspberry)
```console
$ sudo apt-get -y install python-setuptools
$ sudo easy_install pip && sudo pip install docker-compose
```

# Run MMDVMHost, YSFGateway and MMDVM-Dashboard
```console
$ git clone https://github.com/f4hlv/mmdvm-docker.git
$ cd mmdvm-docker
```
Edit docker-compose.yml and run
```console
$ docker network create webgateway
$ docker-compose up -d
```
## Volume
- `/MMDVMHost/MMDVM.ini:ro` Path to the MMDVM.ini File
- `/MMDVMHost/RSSI.dat:ro` Path to the RSSI.dat File (Optional)
- `/MMDVMHost/DMRIds.dat:ro` Path to the DMRIds.dat File (Optional)
- `/YSFClients/YSFGateway/YSFHosts.txt:ro` Path to the YSFHosts.txt File (Optional)

example for MMDVM.ini(raspberry): /home/pi/MMDVM.ini:/MMDVMHost/MMDVM.ini:ro

# Dashboard
Configure options and destination directories `/etc/mmdvm` and `/etc/YSFGateway`. Then delete the setup.php file
## Port
- `80:80` Port configuration. "Port host":80(container port)
## Remove setup.php
```console
$ docker exec mmdvm-dashboard rm /var/www/html/setup.php
```
# Update
```console
$ docker-compose build --no-cache
$ docker-compose up -d
```

# docker-compose
```yml
version: '3'

services:
  mmdvmhost:
    build:
      context: .
      dockerfile: Dockerfile.mmdvmhost
    container_name: mmdvmhost
    volumes:
      - /home/pi/mmdvm-docker/config/MMDVM.ini:/MMDVMHost/MMDVM.ini:ro
#      - /home/pi/mmdvm-docker/config/RSSI.dat:/MMDVMHost/RSSI.dat:ro
#      - /home/pi/mmdvm-docker/config/DMRIds.dat:/MMDVMHost/DMRIds.dat:ro
      - mmdvmhost:/MMDVMHost
    networks:
      mmdvm:
        ipv4_address: 10.10.1.2      
    devices:
      - /dev/ttyACM0:/dev/ttyACM0
    restart: unless-stopped
      
  ysfgateway:
    build:
      context: .
      dockerfile: Dockerfile.ysfgateway
    container_name: ysfgateway
    restart: unless-stopped
    volumes:
      - /home/pi/mmdvm-docker/config/YSFGateway.ini:/YSFClients/YSFGateway/YSFGateway.ini:ro
#      - /home/pi/mmdvm-docker/config/YSFHosts.txt:/YSFClients/YSFGateway/YSFHosts.txt:ro
    depends_on:
      - mmdvmhost
    networks:
      mmdvm:
        ipv4_address: 10.10.1.3
        
  ysf2dmr:
    build:
      context: .
      dockerfile: Dockerfile.ysf2dmr
    container_name: ysf2dmr
    volumes:
      - /home/pi/mmdvm-docker/config/YSF2DMR.ini:/MMDVM_CM/YSF2DMR/YSF2DMR.ini:ro
#      - /home/pi/mmdvm-docker/config/TGList-DMR.txt:/MMDVM_CM/YSF2DMR/TGList-DMR.txt:ro
    depends_on:
      - ysfgateway
    networks:
      mmdvm:
        ipv4_address: 10.10.1.30
    restart: unless-stopped        

  mmdvm-dashboard:
    build:
      context: .
      dockerfile: Dockerfile.mmdvmdashboard
    container_name: mmdvm-dashboard
    restart: always
    ports:
      - "80:80"
    volumes:
      - mmdvmhost:/etc/mmdvm:ro
      - /home/pi/mmdvm-docker/config/MMDVM.ini:/etc/mmdvm/MMDVM.ini:ro
    # - /home/pi/RSSI.dat:/etc/mmdvm/RSSI.dat:ro
    # - /home/pi/DMRIds.dat:/etc/mmdvm/DMRIds.dat:ro
    # - /home/pi/YSFHosts.txt:/etc/YSFGateway/YSFHosts.txt:ro
    pid: host
    restart: always
    networks:
      - webgateway

volumes:
  mmdvmhost:
networks:
  webgateway:
    external:
      name: webgateway
  mmdvm:
    ipam:
      driver: default
      config:
        - subnet: 10.10.1.0/24


```

This software is licenced under the GPL v2 and is intended for amateur and educational use only. Use of this software for commercial purposes is strictly forbidden.
