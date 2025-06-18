# Install
### Download and Unpack the Installer
1. Go to the Harbor releases page.https://github.com/goharbor/harbor/releases
2. Download the online or offline installer for the version you want to install.
3. Use tar to extract the installer package:
   Online installer:
   ```
   bash $ tar xzvf harbor-online-installer-version.tgz
   ```
   Offline installer:
   ```
   bash $ tar xzvf harbor-offline-installer-version.tgz
   ```
### Configure the Harbor YML File
go to directory `harbor` and edit file `harbor.yml.tmpl' and change ***hostname*** to ip address of server, change ***admin password***, and if you do not use https just comment lanes related to ***https***.
and at the end rename the file to `harbor.yml`

### Run the Installer Script
in `harbor` directory, there is a file named `install.sh`. just run this using command `./install.sh`

now using the ip address of the server you can access the harbor web page.

## Login to this registry using docker:

```
docker login <ipAddress>
```

for pushing any image to this registry like what we said in registry document we should create a tag of that image :
```
docker tag <image>:<tag> <ipAddress>/<image>:<tag> 
```
then push the tagged image\
remember that we should login first


____________________________

## Proxy Cache Project

to pull images from harbor and if the image is not in harbor, harbor will pull it from internet and save it in its repository and give it to client.

### Steps:
1. install Harbor on linux
2. you should have v2ray vpn on your ubuntu, use Qv2ray as v2ray client.
3. in Qv2ray go to : Preferences > Inbound Settings
4. in here you should change `listening Address` and `UDP Local IP` from 127.0.0.1 to 0.0.0.0
5. then restart the Qv2ray
6. now you should change the file `docker-compose.yml` in harbor directory.
7. add the following to all services except `jobservices`
   ```
       environment:
        - HTTP_PROXY=http://192.168.163.129:8889
        - HTTPS_PROXY=http://192.168.163.129:8889
        - NO_PROXY=127.0.0.1,localhost,core,jobservice,registry,log,registryctl,postgresql,core,redis,proxy,192.168.163.129
   ```
8. now you should `docker compose -f docker-compose.yml down -v` and then `docker compose -f docker-compose.yml up -d` to start the harbor containers again.
9. to test if the proxy works currect you should first use command `docker ps` to see containers names.
10. look for container which is like `registry` in mine is `goharbor/registry-photon:v2.13.0` and copy its `CONTAINER ID`
11. then using command `docker exec -it <CONTAINER ID> sh` to login to container
12. then use command `curl -x http://192.168.163.129:8889 -L https://registry-1.docker.io/v2/` (which 192.168.163.129 is docker host ip and 8889 is v2ray port):
    if you get {"errors":[{"code":"UNAUTHORIZED","message":"authentication required","detail":null}]} its ok.
13. now you should create a new `project` in harbor web page to pull image from it.
14. first in Harbor web page go to: Administration > Registries > NEW ENDPOINT
   Provider : Docker Hub
15. in Harbor Web page go to : Projects > NEW PROJECT
   activate the Proxy Cache and select the ENDPOINT created previous step\
   lets assume you chose `proxy-dockerhub` as your project name.
16. now you can pull any image (for example busybox) using command `docker pull 127.0.0.1:443/proxy-dockerhub/busybox:latest` and it will download it from internet and save it to harbor and you host which you can see it by `docker images`

### ATTENTION:
*i wanted to change the vpn server from 192.168.163.129 (127.0.0.1) to 192.168.163.1 which is another server in lan*\
*i changed the `docker-compose.yml` file and added:*
```
environment:
  - HTTP_PROXY=http://192.168.163.129:8889
  - HTTPS_PROXY=http://192.168.163.129:8889
  - NO_PROXY=127.0.0.1,localhost,core,jobservice,registry,log,registryctl,postgresql,core,redis,proxy,192.168.163.129
```
*and used commands:*
```
docker compose -f docker-compose.yml down -v
docker compose -f docker-compose.yml up -d
```
*but still when i tried to login to harbor using command:*
```
docker login 192.168.163.129:443
```
*i got error :*
```
Error response from daemon: Get "https://192.168.163.129:443/v2/": proxyconnect tcp: dial tcp 127.0.0.1:8889: connect: connection refused
```
*it means docker was still using the 127.0.0.1:8889 as proxy*\
*to check first i exec into `harbor-core` container using command :*
```
docker exec -it <CONTAINER ID> sh
```
*and used commands:*
```
echo $HTTP_PROXY
echo $HTTPS_PROXY
```
*and for both i got:*
```
http://192.168.163.1:10811
```
*it means here the proxy changed.*\
*so i had to change something else, actually the docker daemon was using the 127.0.0.1:8889 as proxy. so i edited the file `/etc/systemd/system/docker.service.d/http-proxy.conf` from:*
```
[Service]
[Service]
Environment="HTTP_PROXY=http://192.168.163.129:8889"
Environment="HTTPS_PROXY=http://192.168.163.129:8889"
```
*to:*
```
[Service]
Environment="HTTP_PROXY=http://192.168.163.1:10811"
Environment="HTTPS_PROXY=http://192.168.163.1:10811"
```
*after that i used commands:*
```
sudo systemctl daemon-reload
sudo systemctl restart docker
```
*then to verify my changes:*
```
sudo systemctl show --property=Environment docker
```


### Another Problem:
*i tried to login using command :*
```
docker login 192.168.163.129:443
```
*then i got:*
```
Error response from daemon: Get "https://192.168.163.129:443/v2/": tls: failed to verify certificate: x509: certificate relies on legacy Common Name field, use SANs instead
```
***to fix this issue :***
*in `harbor` directory i created a directory named `certs`:*
```
cd /your/harbor
mkdir certs
```
*then i created a file inside this directory named "cert.conf" which inside the file was like:*
```
[req]
default_bits = 2048
prompt = no
default_md = sha256
distinguished_name = dn
x509_extensions = v3_req

[dn]
C = DE
ST = State
L = City
O = MyOrg
CN = 192.168.163.129

[v3_req]
subjectAltName = @alt_names

[alt_names]
IP.1 = 192.168.163.129
```
*then generated the certificate using command:*
```
openssl req -x509 -nodes -days 365 \
  -newkey rsa:2048 \
  -keyout harbor.key \
  -out harbor.crt \
  -config cert.conf
```
*then i edited `harbor.yml` file and changed lines refer to https to below:*
```
hostname: 192.168.163.129
https:
  port: 443
  certificate: /path/to/harbor/certs/harbor.crt
  private_key: /path/to/harbor/certs/harbor.key
```
*then i copied the `harbor.crt` to docker certs directory:*
```
sudo mkdir -p /etc/docker/certs.d/192.168.163.129:443/
sudo cp ./certs/harbor.crt /etc/docker/certs.d/192.168.163.129:443/ca.crt
sudo systemctl restart docker
```
*make sure you restart the docker service (last command)*\
*then in `harbor` directory i used command `./prepare` then i edited the `docker-compose.yml` file and added:*
```
environment:
  - HTTP_PROXY=http://192.168.163.129:8889
  - HTTPS_PROXY=http://192.168.163.129:8889
  - NO_PROXY=127.0.0.1,localhost,core,jobservice,registry,log,registryctl,postgresql,core,redis,proxy,192.168.163.129
```
*for services and used commands:*
```
docker compose down -v
docker compose up -d
```
*after this i could login and pull from harbor*
