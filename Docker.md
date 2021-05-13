# Docker 軟體貨櫃  

---

![](https://i.imgur.com/DMoTc40.png)  

---
## :ab: 實作Docker   
```
$ sudo apk add docker
```
:warning: 下載之後重要三步驟  
```
$ sudo rc-update add docker boot  
$ sudo addgroup bigred docker
$ sudo  reboot
```
:earth_africa: **Docker 指令集**  
\$ docker info  
(查看套件資訊)  

\$ docker run  
\( 產生 containd )  
參數 :  
\-d : 背景執行  
\-h : containd 電腦名稱  
\-m : 限制記憶體  
\-f : 在前景執行(在指令裡面打的)  
\-it : 產生終端機(有貝殼程式就一定要有)  
\- -net : 設定使用的網型態  
\- -rm : containd 停止後立即自動刪除  
\- -name : 給 containd 的名字   
\- -cpus : 設定只能使用幾 % 的 cpu  
\- -cpuset-cpus : 設定使用的 core  
```
$ docker run --rm --name c1 --cpuset-cpus="1" --cpus="0.2"  -itd  busybox  yes
```

\$ docker ps -a   
(查看所有 containd 狀況)  

\$ docker start  
(啟動 containd )   

\$ docker stop  
(停止 containd )  

\$ docker rm  
(刪除 containd ，-f 為強制刪除)  

\$ docker rmi  
(刪除 image 檔)  

\$ docker stats  
(查看 containd 檔使用資源)  

\$ docker exec  
(對自己存在的 containd 下達指令)  
```
docker exec c1 hostname -i
docker exec -it c1 sh
```




:cd: **關於光碟的指令**  

\$ docker pull  
(下載光碟)  

\$ docker commit  
(把 image 檔壓成光碟)  

\$ docker login   
(登入 docker hub )  

\$ docker push \<Login Name\>/a1  
(推光碟上到光碟小舖)  
\$ docker logout
(登出 docker hub )  

:e-mail: 關於網路的指令  

\$ docker  network  ls  
(查看有什麼網路類型)  
基本三種:  
1. Bridge mode (使用虛擬橋接器)  
2. Host mode (使用主機所有網路介面)(不建議使用)  
3. None (沒有網路介面)(但我們可以自己塞一張)  

\$ brctl show  
(看虛擬橋接器 Docker0 使用的網卡)  

 ### 新增 Docker 網路
![](https://i.imgur.com/yaeaFCg.png)  


---

### :a: Containd  
![](https://i.imgur.com/EAvNbGI.png)  
```
$ sudo apk add containerd
$ containerd -v
$ sudo containerd &
```
(下載containd，查看版本，背景執行)  

![](https://i.imgur.com/MRyB5D1.png)  



---
## :eagle: 自製網卡  
```
$ sudo modprobe tun

$ sudo tunctl -b -u bigred
tap0

$ ifconfig tap0
tap0: flags=4098<BROADCAST,MULTICAST>  mtu 1500
        ether 62:da:85:b2:1f:81  txqueuelen 1000  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

```
***(-b : 創造網路卡，-u : 給專屬的使用者)***  

:+1: **dknet 是老師自己寫的程式(塞網卡用的)**  
```
$ mkdir bin
$ echo $'#!/bin/bash
[ "$#" != 2 ] && echo "dknet ctn net" && exit 1

ifconfig $2 &>/dev/null
[ "$?" != "0" ] && echo "$2 not exist" && exit 1

x=$(docker inspect -f \'{{.State.Pid}}\' $1 2>/dev/null)
[ "$x" == "" ] && echo "$1 not exist" && exit 1

[ ! -d /var/run/netns ] && sudo mkdir -p /var/run/netns

if [ ! -f /var/run/netns/$x ]; then
   sudo ln -s /proc/$x/ns/net /var/run/netns/$x
   sudo ip link set $2 netns $x
fi

exit 0
' > bin/dknet ; chmod +x bin/dknet

```
### :yin_yang: 給虛擬網卡(真實網卡)  
```
$ docker run --rm --name d1 --cap-add=NET_ADMIN --net='none'  -itd busybox  sh
3879c04ac6435e5cd0a64effdc5843bce57c3b1e4f277722806f622af53d4425

將 tap0 網卡加入 d1 貨櫃主機的 Network Namespace
$ dknet d1 tap0(eth0)

在 ctn 主機中,  此時看不到 tap0 網卡
$ ifconfig tap0(eth0)
tap0: error fetching interface information: Device not found

$ docker exec -it d1 sh
/ # ifconfig -a
tap0      Link encap:Ethernet  HWaddr 00:0C:29:4A:69:10  
          BROADCAST MULTICAST  MTU:1500  Metric:1
          RX packets:658 errors:0 dropped:0 overruns:0 frame:0
          TX packets:213 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:747802 (730.2 KiB)  TX bytes:28428 (27.7 KiB)

lo        Link encap:Local Loopback  
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0 
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)


[重要] ifconfig 必須有 "-a" 參數才能看到 tap0 
```
***(給真實網卡的話一開始就不用自己製作囉^^)***

### :black_square_button: 設定網卡資訊  
```
/ # ifconfig  tap0(eth0)  192.168.55.66  netmask  255.255.255.0
/ # ifconfig tap0(eth0)
tap0    Link encap:Ethernet  HWaddr FA:3D:33:4F:2B:AC  
          inet addr:192.168.55.66  Bcast:192.168.55.255  Mask:255.255.255.0
          UP BROADCAST MULTICAST  MTU:1500  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)

/ # route add default gw 192.168.55.1
/ # exit

$ docker rm -f d1

[註] 移除 d1貨櫃主機, 被獨佔的網卡, 會一並被刪除  

```

---
### :b: Runc  
![](https://i.imgur.com/9s0INUH.png)  
```
$ sudo apk update &>/dev/null ; sudo apk add runc  
$  sudo chown root:root -R rootfs/
```
因 runc 命令內定執行帳號為 root, 而 rootfs 目錄的 Owner 是 bigred, 所以需將 rootfs 目錄的 owner 改為 root  

:arrow_right: 產生 OCI 標準的 Container 設定檔  
![](https://i.imgur.com/SyeqKEc.png)  

:arrow_right: 建立 bb8 containd  
```
$ sudo runc run bb8
```
:arrow_right: 保護 Linux Kernel 系統目錄  

![](https://i.imgur.com/VblPHeL.png)  




---

### :yellow_heart: docker公司的歷史演進  
![](https://i.imgur.com/JVcca4X.png)  
:warning: ***Enterprise 年年收***  


---

# Dockerless
:question: 因為Daemon 太多，成為安全的破口，且docker engine 權力又很大，一旦被攻擊就很危險了  


---
:8ball: **k8s進化史**
![](https://i.imgur.com/9o4CQ3s.png)


---
:earth_asia: 各軟體的時間軸  
 ![](https://i.imgur.com/PaJgHd8.png)  
 

---
## :rabbit: Docker 的自我解救辦法  
![](https://i.imgur.com/FrcBgkX.png)  


---

                  



