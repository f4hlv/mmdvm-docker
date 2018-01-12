# mmdvm-docker (BETA image)

# Install Docker
```console
$ curl -fsSL get.docker.com -o get-docker.sh
$ sudo sh get-docker.sh
```

# Install docker-compose
## (Debian) ##
```console
$ curl -L https://github.com/docker/compose/releases/download/1.18.0/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
$ chmod +x /usr/local/bin/docker-compose
```

## (Raspberry) ##
```console
$ apt-get install python3-pip
$ pip3 install docker-compose
```

# Run MMDVMHost, YSFGateway and MMDVM-Dashboard
```console
$ git clone https://github.com/f4hlv/mmdvm-docker.git
$ cd mmdvm-docker

### Example ###
$ docker-compose up -d
```
