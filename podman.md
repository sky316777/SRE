# PODMAN


---
![podman](https://i.imgur.com/gGgXmK8.png)  

### podman 運作架構解說  
**podman 是 program 不是daemon(一直在背景執行)執行後變process**  
* 透過skopeo抓images檔(可以處理很多不同版本)  
有2派: Docker or OCI組織  
* buildah 執行 Dockerfile (做成image)  
* 產生 container  
crun: 用C語言  
runC: 用go語言  
執行速度上crun比runc快一倍  

### 安裝 podman
```
$ cat /etc/os-release 
(此檔案做了很多環境變數的宣告)  
NAME="Ubuntu"
VERSION="20.04.2 LTS (Focal Fossa)"
ID=ubuntu
ID_LIKE=debian
PRETTY_NAME="Ubuntu 20.04.2 LTS"
VERSION_ID="20.04"
HOME_URL="https://www.ubuntu.com/"
SUPPORT_URL="https://help.ubuntu.com/"
BUG_REPORT_URL="https://bugs.launchpad.net/ubuntu/"
PRIVACY_POLICY_URL="https://www.ubuntu.com/legal/terms-and-policies/privacy-policy"
VERSION_CODENAME=focal
UBUNTU_CODENAME=focal
#CODENAME = 小名


$ source  /etc/os-release

$ sudo sh -c "echo 'deb http://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/xUbuntu_${VERSION_ID}/ /' > /etc/apt/sources.list.d/devel:kubic:libcontainers:stable.list"
(加下載點到Ubuntu 的 source.list.d的清單中)

$ wget -nv https://download.opensuse.org/repositories/devel:kubic:libcontainers:stable/xUbuntu_${VERSION_ID}/Release.key -O- | sudo apt-key add -
(將公鑰加到UB系統中)

$ sudo apt update; sudo apt install podman -y

```

**Docker 只能用docker的光片中心;
Podman 透過skopeo都可以用各種不同的images**
```
$ sudo nano /etc/containers/registries.conf
(光片中心有兩個讓我們做選擇，其中quay.io 是 ret hat 創造的)  
......
unqualified-search-registries = ["docker.io","quay.io"]
......

$ sudo nano /etc/containers/registries.conf.d/000-shortnames.conf
(不同的名字就會去到該網站的光片中心去做光碟片下載)

[aliases]
  # centos
  "centos" = "quay.io/centos/centos"
  # containers
  "skopeo" = "quay.io/skopeo/stable"
  "buildah" = "quay.io/buildah/stable"
  "podman" = "quay.io/podman/stable"
  # docker
  "alpine" = "docker.io/library/alpine"
  "docker" = "docker.io/library/docker"
  "registry" = "docker.io/library/registry"
  "hello-world" = "docker.io/library/hello-world"
  "swarm" = "docker.io/library/swarm"
  # Fedora
  "fedora-minimal" = "registry.fedoraproject.org/fedora-minimal"
  ........

```

```
(同docker run功能，後面接光碟片name，這是podman的test command, 看能否正常建立container)  
$ podman run hello-world
Resolved "hello-world" as an alias (/etc/containers/registries.conf.d/000-shortnames.conf)
Trying to pull docker.io/library/hello-world:latest...
Getting image source signatures
Copying blob 0e03bdcc26d7 done  
Copying config bf756fb1ae done  
Writing manifest to image destination
Storing signatures

Hello from Docker!
This message shows that your installation appears to be working correctly.
............

$ podman ps -a
(目前關機中)
CONTAINER ID  IMAGE                                 COMMAND  CREATED         STATUS                     PORTS   NAMES
700b68c4e769  docker.io/library/hello-world:latest  /hello   27 seconds ago  Exited (0) 27 seconds ago          nice_raman

$ podman start 700b
(啟動)
$ podman ps -a
(看還是關機中)
$ podman history IMAGE的名字(docker.io/library/hello-world:latest)

*偷天換日的做法，擇一做使用*
方法1:
$ sudo nano /etc/profile (所有使用者都可以用)
alias docker=‘sudo podman’

方法2:
或是用
$ sudo nano .bashrc (只 for bigred這個帳號)
alias docker=‘sudo podman’


$ podman rm 700b
700b68c4e769e2ad2f2371f112fc3c612c099683139b3ba62cf7b1cb5e412d19
```

---

### Podman Rootful Container

**rootful 只能用root代表操作**

```
$ echo "alias docker='sudo podman'" > .bashrc

$ alias docker='sudo podman'

$ docker run -it  alpine sh
Resolved "alpine" as an alias (/etc/containers/registries.conf.d/000-shortnames.conf)
Trying to pull docker.io/library/alpine:latest...
Getting image source signatures
Copying blob ba3557a56b15 done  
Copying config 28f6e27057 done  
Writing manifest to image destination
Storing signatures
fc115717b17c4288b3618368f216d5fa5c170e7b08bcd3ac26fd1a5cb3717b01

```

```
$ docker exec -it fc11 sh
/ # whoami
root

/ # hostname -i
10.88.0.3

/ # route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0           10.88.0.1       0.0.0.0         UG    0      0        0 eth0

/ # ping -c 2 www.hinet.net
PING www.hinet.net (114.25.250.8): 56 data bytes
64 bytes from 114.25.250.8: seq=0 ttl=42 time=32.628 ms
64 bytes from 114.25.250.8: seq=1 ttl=42 time=47.885 ms

/ # cat /etc/resolv.conf 
nameserver 172.16.119.1

/ # exit
```

:arrow_right: 特別說明:  
\$ ping -c 2 www.hinet.net  
現在大多大型企業都在使用ipv6，但虛擬機沒有在使用ipv6時會導致ping不到那些有ipv6企業的網站，所以做法就是把vm的ipv6功能關閉，這樣在ping的時候就只會抓該網站的ipv4網址，ping才可以成功  
作法如下:  
```
$ sudo nano /etc/default/grub

改裡面這一行
GRUB_CMDLINE_LINUX="ipv6.disable=1"
$ sudo update-grub
$ reboot
```

### Podman Rootful Container Network

**Docker自建網路可以用名字互相ping，   
但是podman rootful的自建網路不行，只能用ip互相ping**

```
$ sudo apt install bridge-utils
(有裝才可以用brctl show)

$ brctl show
bridge name	bridge id		STP enabled	interfaces
cni-podman0	8000.86e232192730	no		vethbbf4d850

$ docker network create --driver=bridge --subnet=192.168.166.0/24 --gateway=192.168.166.254  mynet
/etc/cni/net.d/mynet.conflist

$ cat /etc/cni/net.d/mynet.conflist | grep subnet
                     "subnet": "192.168.166.0/24",

(第一次建立 Podman 網路，不會建立虛擬橋接器，需產生連接 mynet 的 Container 才會產生虛擬橋接器，所以cni-podman1在一開始並不會顯示出來喔!)
$ brctl show
bridge name	bridge id		STP enabled	interfaces
cni-podman0	8000.86e232192730	no	       vethbbf4d850
		
$ docker run --rm --net mynet --name a1 -h cg61 -d alpine sleep 360

$ docker run --rm --net mynet --name a2 -h cg62 -d alpine sleep 360

(因為上面有產生2個用mynet的container，所以在下面的cni-podman1 的 interfaces 才會有2筆資料)
$ brctl show
bridge name	bridge id		STP enabled	interfaces
cni-podman0	8000.4e7b314d9b71	no	        veth67298255
cni-podman1	8000.f695873fdba7	no	        veth0e1577bc
						        vethdef97e4b

$ docker exec a2 ping -c 2 192.168.166.6
PING 192.168.166.6 (192.168.166.6): 56 data bytes
64 bytes from 192.168.166.6: seq=0 ttl=42 time=0.083 ms
64 bytes from 192.168.166.6: seq=1 ttl=42 time=0.125 ms

$ docker exec a2 ping -c 2 cg61
ping: bad address 'cg61'

$ docker stop a1 a2

```

---

### Podman Rootless Container  
**rootless 一般使用者帳號就可以操作且都會有自己獨立的環境可以創造container系統**

```
$ mkdir html; echo "<h1>Rootless Container</h1>" > html/index.html
(index.html是網站的首頁檔)

$ podman run --rm -d --publish 8080:80 --volume ${PWD}/html:/usr/share/nginx/html nginx
✔ docker.io/library/nginx:latest
Trying to pull docker.io/library/nginx:latest...
.........
5d6265214ef71ffce826d38b16dffd2557a0cfcd353dbecef92b87d4a6d47071

$ curl http://localhost:8080
<h1>Rootless Container</h1>

$ podman exec 5d62 whoami
root 
$ ps aux | grep -v grep | grep nginx
bigred     10709  0.0  0.0  10632  5732 ?        Ss   21:51   0:00 nginx: master process nginx 
-g daemon off;
100100     10726  0.0  0.0  11024  2512 ?        S    21:51   0:00 nginx: worker process

$ podman stop 5d62

(自己使用者創建的一切，都在自己家目錄下的.local)
$ tree -L 4 .local
.local
└── share
    └── containers
        └── storage
            ├── cache
            ├── libpod
            ├── mounts
            ├── overlay
            ├── overlay-containers
            ├── overlay-images
            ├── overlay-layers
            ├── storage.lock
            ├── tmp
            └── userns.lock
```
參數說明:  
--publish : -p  
--volume : -v  
\${PWD}/html是UBS的目錄，啟動時掛載到專門開nginx的container目錄(/usr/share/nginx/html nginx)  
html nginx 是imges檔


:arrow_right: 用bigred創造rbean/gbean後登入，
現在登入gbean打指令  
**(切換使用者不可以用su gbean，因為這個指令不會跑/etc/profile，要用exit再重新ssh gbean)**
\$ podman ps –a  
會發現不會有rbean的container，每個使用者有自己獨立的環境  
\$ mkdir html; echo "\<h1>Rootless Container\</h1>" > html/index.html
\$ podman run --rm -d --publish 8081:80 --volume \${PWD}/html:/usr/share/nginx/html nginx
**(因為rbean跟gbean都是用US2004開出來的帳號，8080是rbean用的，所以2台port號不可以一樣)**


```
$ podman run --rm -it alpine
Resolved "alpine" as an alias (/etc/containers/registries.conf.d/000-shortnames.conf)
Trying to pull docker.io/library/alpine:latest...
.........
/ # whoami (當前使用者帳號)
root
/ # hostname -i
10.0.2.100
/ # cat /etc/resolv.conf
nameserver 10.0.2.3
nameserver 172.16.119.1
/ # exit
```

:arrow_right: 使用一般使用者帳號創造container，看外面主機是哪個帳號run這個container就是該使用者; 但container裡面還是root，且因為capability，所以此root跟我們一般強大的root是不一樣的)  


```
$ podman run --rm -itd --name a1 alpine
$ podman run --rm -itd --name a2 alpine

(整個網路系統是使用slirp4網路架構(撥號網路系統)，所以每一個container IP位置是相同的，大家不能互ping)  
$ podman exec a1 hostname -i
10.0.2.100
$ podman exec a2 hostname -i
10.0.2.100

$ ps aux | grep -v grep | grep -e "^rbean.*slirp"
rbean      30340  0.0  0.0   2576  1712 pts/0    S    00:20   0:00 /usr/bin/slirp4netns --disable-host-loopback --mtu=65520 --enable-sandbox --enable-seccomp -c -e 3 -r 4 --netns-type=path /run/user/1001/netns/cni-11d24834-eb38-0a74-903d-55a92ede2a5a tap0
rbean      30450  0.0  0.0   2576  1716 pts/0    S    00:21   0:00 /usr/bin/slirp4netns --disable-host-loopback --mtu=65520 --enable-sandbox --enable-seccomp -c -e 3 -r 4 --netns-type=path /run/user/1001/netns/cni-04866607-0f3c-a5c9-0222-12d25b96c258 tap0

$ podman rm -f a1 a2
```


```
$ podman network create --driver=bridge --subnet=192.168.188.0/24 --gateway=192.168.188.254  mynet2
/home/rbean/.config/cni/net.d/mynet2.conflist  
(rootless可以建立網路但沒有虛擬橋接器(bridge)，建立虛擬橋接器不可以用普通使用者創建虛擬橋機器，必須是要用外部強大的root權限才可以創建)  

$ podman network ls
(確認網路是否有建立成功)  
NAME    VERSION  PLUGINS
mynet2  0.4.0    bridge,portmap,firewall,tuning,dnsname

$ cat /home/rbean/.config/cni/net.d/mynet2.conflist | grep '"subnet":' | tr -d ' '
"subnet":"192.168.188.0/24",

$ podman run --name n1 --net mynet2 -itd alpine
$ podman run --name n2 --net mynet2 -itd alpine

(由mynet2網路發放的ip，這樣就是2個ip)
$ podman exec n1 hostname -i
192.168.188.1
$ podman exec n2 hostname -i
192.168.188.2

(podman rootful 的自建網路是無法利用電腦名稱互ping的)
$ podman exec n2 ping -c2 n1
PING n1 (192.168.188.1): 56 data bytes
64 bytes from 192.168.188.1: seq=0 ttl=42 time=1.708 ms
64 bytes from 192.168.188.1: seq=1 ttl=42 time=0.135 ms

```
```
$ podman ps -a
CONTAINER ID  IMAGE                                                                                                      COMMAND         CREATED        STATUS            PORTS   NAMES
3037f4532112  docker.io/library/alpine:latest                                                                            /bin/sh         3 minutes ago  Up 3 minutes ago          n1
2afb95834b2f  quay.io/libpod/rootless-cni-infra@sha256:adf352454666f7ce9ca3e1098448b5ee18f89c4516471ec99447ec9ece917f36  sleep infinity  3 minutes ago  Up 3 minutes ago          rootless-cni-infra
e62b28560518  docker.io/library/alpine:latest                                                                            /bin/sh         3 minutes ago  Up 3 minutes ago          n2

(rootless-cni-infra : 中華電信的撥號網路機房(透過電話線)，裡面會啟動很多個slirp服務，會發送給container不同ip)
```

---

### Podman Rootless Container & Pod  
**Pod manager (management) 就是管理 pod**  

* pod裡面由多個container組成，且使用環境很安全因為不容易斷線，畢竟是共用這個pod的這一個網路介面  
* K8s/k3s 運作的最小單元就叫做pod

![pod](https://i.imgur.com/ehuVDdX.png)

```
$ podman pod create -n mypod
(-n: name，產生pod名字叫做mypod)

$ podman pod list
POD ID        NAME    STATUS   CREATED         INFRA ID      # OF CONTAINERS
6ba66538a764  mypod   Created  54 seconds ago  22e55cf8d3ec  1

$ podman ps -a --pod
(ps -a --pod : 這個container屬於哪個pod)

CONTAINER ID  IMAGE                            COMMAND  CREATED             STATUS                    PORTS   NAMES               POD ID        PODNAME
.......                              
22e55cf8d3ec  k8s.gcr.io/pause:3.2                      About a minute ago  Created                           6ba66538a764-infra  6ba66538a764  mypod

$ podman run -dt --pod mypod alpine top
(產生出來的container要放到mypod中)

$ podman ps --pod | grep mypod
22e55cf8d3ec  k8s.gcr.io/pause:3.2                      15 minutes ago  Up 3 minutes ago          6ba66538a764-infra   6ba66538a764  mypod
cdcb45130dae  docker.io/library/alpine:latest  top      3 minutes ago   Up 3 minutes ago          nervous_ardinghelli  6ba66538a764  mypod

$ podman run -d --pod mypod nginx
(共有3個container (pause/alpine/nginx))

$ podman exec -it cdcb sh
/ # apk add curl
/ # curl http://localhost
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
..........

(因為Container共用網路卡，所以登入alpine去curl自己的ip就能看到在同一個pod裡的container起的網站，主要是想證明同一個pod裡是共用網路介面)

/ # exit

```
 

