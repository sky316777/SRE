# Docker 軟體貨櫃  

---

![d1](https://i.imgur.com/SiVKtol.jpg)  

### Docker 運作架構解說 :  
1. sh/bash : 開啟貝殼程式  
2. Docker Daemon : 提供使用者高階命令，讓系統執行 application container  
3. containerd : 負責控管 container的一生(建立 log/開機/關機/移除)，**產生rootfs&container.json**  
4. shim : 墊片，負責監控 container 的所有行為，並回報資訊給 containerd  
5. runC : **使用rootfs&container.json去產生** container。根據國際標準OCI寫出來的程式，只會負責產生 container，產生完就離開。**產生 container 主要有6項隔離技術**  

### Container架構解說 :  
1. Server : hardware  
2. Host OS : Kernel  
3. App : programe  
4. Libs : 相依檔 (.so= system object / .ko= kernel object / command)   
5. Bins : linux others command (Ex. yes/sleep)  
6. **application container : 把一個 Programe 隔離出來變成一台電腦，虛擬出來的電腦共用kernel**，所以沒有開機程序，主要是下面的 kernel 啟動就代表啟動開機程序，也沒有關機程序>> 啟動速度非常快  

-----

![d2](https://i.imgur.com/rNXTpmJ.png)

### Docker 運作架構解說 :  
Docker image center  
1. pull: 從光碟小舖download docker image (CD)  
2. runC 產生container  
3. 變成一台電腦，運作  
4. commit: 燒成光碟  
5. push: 上傳到docker的光碟片中心(備份)  

*backup.tar: 有些公司無法讓員工上網，所以無法連上網到docker光片中心，所以先把檔案save成tar檔，放入USB後讓企業內部用load還原變成image，做光碟片的分享  
*Dockerfile: 如何做出光碟片的步驟

---

## :page_facing_up: 目錄 :  
1. containerd  
2. runc  
3. namespace  
4. chroot  
5. overlay2  
6. cgroup  
7. slirp4netns   
8. Docker
9. Docker 橋接網路  
10. Dockerfile  
11. Docker Container的備份與還原  


---

## :bookmark: Containerd  

![contnainerd](https://i.imgur.com/EAvNbGI.png)  

:arrow_right: 下載containerd，查看版本，背景執行
```
$ sudo apk add containerd
$ containerd -v
$ sudo containerd &
```

:arrow_right: Containerd 原生管理命令 - ctr  
(下載 Container Image，開啟- -tty 虛擬終端機跑程式)
```
$ sudo ctr image pull docker.io/library/busybox:latest
$ sudo ctr run --tty docker.io/library/busybox:latest b1 sh

/ # ps aux
PID   USER     TIME  COMMAND
    1 root      0:00 sh
    8 root      0:00 ps aux
/ # exit

$ sudo ctr container list
CONTAINER    IMAGE                                           RUNTIME                  
b1                 docker.io/library/busybox:latest        io.containerd.runc.v2 

$ sudo ctr container rm b1
$ sudo ctr image remove docker.io/library/busybox:latest docker.io/library/busybox:latest
```

---

## :boom: Runc  
**透過 RunC 建立 Container 電腦，需準備Container的設定檔&檔案系統目錄(rootfs)**

下載 runc，因為 runc 內定執行帳號為 root，而 rootfs 目錄的 owner 是 bigred，所以需將 rootfs 目錄的 owner 改為 root  
```
$ sudo apk update &>/dev/null ; sudo apk add runc 
$ sudo chown root:root -R rootfs/
```
:arrow_right: 產生 OCI 標準的 Container 設定檔  
![runc](https://i.imgur.com/DeNfvN3.png)  

:arrow_right: 保護 Linux Kernel 系統目錄，所以無法刪除  
```
$ sudo runc run bb8
/# cat /proc/meminfo | head -n 3
MemTotal:        6088832 kB
MemFree:         5408756 kB
MemAvailable:    5738776 kB

/ # rm /proc/meminfo
rm: remove '/proc/meminfo'? y
rm: can't remove '/proc/meminfo': Permission denied
```

---

## namespace  
**用來隔離不同 Container 的執行空間**

:star: 要做出namespace需要有以下隔離技術:  
cgroup
ipc(inter process communication): container之間的溝通管道(透過記憶體)  
mnt(mount)  
net: 自己獨立的網路系統  
pid: 自己的process系統  
user: chroot的帳號自己獨立作業  
uts: hostname  

sudo unshare  
參數 :  
\- -uts : 隔離 hostname   
\- -pid : 隔離 process  
\- -fork : 與 --pid 一起使用，讓其可產生自己的子目錄  
\- -mount-proc : 蓋掉原本的資訊(但原本的資訊還在)   
\- -net : 只創造網路空間  
-R : 改變根目錄(chroot)  
```
$ sudo unshare --pid --fork --mount-proc --uts -R r2d2/rootfs sh
```

**ps所有命令資料一定都從/proc(存在記憶體)來的**，雖然啟用 PID Namespace，ps 命令還是會看到Host (ctn)的所有 Process 資訊，這是因爲/proc的資料夾還在bigred裡面, 並不是在root，代表process的資訊並沒有被隔離出來喔!  
```
$ sudo unshare --pid --fork --uts sh
$ pstree -p 
init(1)-+-acpid(4048)
        |-chronyd(4153)
        |-crond(4180)
        |-getty(4292)
        |-getty(4295)
        |-getty(4300)
        |-getty(4304)
        |-getty(4308)
        |-login(4290)---bash(4313)---dialog(4355)
        |-sshd(4211)---sshd(4648)---sshd(4653)---bash(4654)---unshare(5695)---sh(5696)---pstree(5714)
        |-syslogd(3994)
        `-udhcpc(3934)
```


---

## chroot (設給program)  
舊名sandbox(沙箱)，**針對執行bin/bash程式的目錄，改變它的根目錄**  

1.  mkdir rootfs; cd rootfs; curl -s -o alpine.tar.gz http://dl-cdn.alpinelinux.org/alpine/v3.13/releases/x86_64/alpine-minirootfs-3.13.4-x86_64.tar.gz;  
:arrow_right: 創造一個資料夾 rootfs，然後在裡面下載 alpine 根目錄下的所有檔案  
參數解說:  
curl : 砍網頁程式碼(似爬蟲)/下載網站中的檔案  
tar=把很多檔案打包變成一個檔案(打包檔)  
gz=壓縮(gzip壓縮演算法)  
2.  tar xvf alpine.tar.gz &>/dev/null ;rm alpine.tar.gz; cd ..   
:arrow_right: xvf : 解壓縮參數(記得刪掉壓縮檔，才不會一直堆積東西)  
3.  sudo chroot rootfs/ /bin/sh  
:arrow_right: 把 rootfs 更改為在根目錄下執行，且使用的是 /bin/sh 貝殼程式(進去後可使用 \$whoami 檢查)  

### 指定要更改的使用者(不要root)
```
bigred@lcs016:~$ sudo chroot --userspec=bigred:bigred  rootfs/ /bin/sh  
@lcs016:/$ whoami  
whoami: unknown uid 1000
@lcs016:/$ adduser -s bigred
adduser: permission denied (are you root?)
@lcs016:/$ exit
```
:arrow_right: 這代表說chroot中根本沒有bigred這個名字，所以系統先發一個noname給這個使用者。隨意登入的使用者是不能新增使用者的，要先用root權限登入創造這個bigred名字的使用者, 這樣下這行指令系統才能真的對應到  
參數解說:  
--userspec : 指定使用者的參數  
:warning:  **進去後會沒有使用者，因為是在 rootfs 資料夾裡原始的 alpine  
所以記得要先進去使用 root 權限創造你要的使用者**  
```
bigred@lcs016:~$ sudo chroot rootfs/ /bin/sh
root@lcs016:/$ adduser -s /bin/bash bigred
Changing password for bigred
root@lcs016:/$ exit
bigred@lcs016:~$ sudo chroot --userspec=bigred:bigred rootfs/ /bin/sh
bigred@lcs016:/$ whoami
bigred
```

---

## overlay2  
**是堆疊型的檔案系統(union file system)**

![overlay2](https://i.imgur.com/B4LGLJS.png)  
merged = upper+lower的組合, 掛載overlay2的資料夾
由人類視角就是由上往下看東西  
當我們使用merged做事時, 實際上是使用到overlay2在做事  

:wave: 
重點提醒:  
Union file system聯合檔案系統(只有兩層)  
常用狀況為 :   
upper 放 USB(硬碟)  
lower 放 CD  

1.  $ mkdir upper lower merged work  
(先創造要使用的目錄和檔案)
```
$ echo "I'm from lower!" > lower/in_lower.txt  
$ echo "I'm from upper!" > upper/in_upper.txt  
$ echo "I'm from lower!" > lower/in_both.txt  
$ echo "I'm from upper!" > upper/in_both.txt  
```
 
2. \$ sudo mount -t overlay overlay -o lowerdir=/home/bigred/lower,upperdir=/home/bigred/upper,workdir=/home/bigred/work  /home/bigred/merged  
**(mount -t overlay : 掛載三個資料夾到merged，wokdir: 運作時的工作目錄，上下兩層的暫存區)**  
3.  **發現both其實是看到upper的，因為lower的被擋住了**  
```
$ cat merged/in_both.txt  
I'm from upper!  
```

4. **加入到 merged 的檔案也會同時加到 upper**
```
$ echo 'new file' > merged/new_file  
$ ls -l \*/new_file  
-rw-r--r-- 1 bigred bigred 9 Apr 29 23:16   merged/new_file  
-rw-r--r-- 1 bigred bigred 9 Apr 29 23:16   upper/new_file  
```
  
5. 刪掉 both後會發現，**merged 的確實被刪掉，因為刪的both是來自upper的檔案**，該檔案敘述變成C，表示檔案已經被刪掉, 但是系統還是會顯示；**lower 的both還會存在，因為該檔案是read only**  

```
\$ rm merged/in_both.txt  
\$ ls -al merged/  
........  
-rw-r--r--  1 bigred bigred   16 Apr 29 23:12 in_lower.txt  
-rw-r--r--  1 bigred bigred   16 Apr 29 23:13 in_upper.txt  
-rw-r--r--  1 bigred bigred    9 Apr 29 23:16 new_file  
\$ ls -al upper/  
.......
c---------  1 root   root   0, 0 Apr 29 23:17 in_both.txt  
-rw-r--r--  1 bigred bigred   16 Apr 29 23:13 in_upper.txt  
-rw-r--r--  1 bigred bigred    9 Apr 29 23:16 new_file
```  

6. **想取消 mount 就打 umount 指令**  
```
$ sudo umount merged/  
$ ls -al merged/  
total 8  
drwxr-sr-x  2 bigred bigred 4096 Apr 29 23:12 .  
drwxr-sr-x 12 bigred bigred 4096 Apr 29 23:12 ..
```    

---

## cgroup  
**用來限制process的運算資源，分配資源(cpu,core,memory)**  
**(在cgroup之下創建的目錄會繼承上一層目錄的資料)**

1. 控制群組使用memory的資源  
\$ sudo mkdir /sys/fs/cgroup/memory/demo    
 **(以下重要)**  
```
$ nano memlimit.sh   
#!/bin/bash  
[ ! -d /sys/fs/cgroup/memory/demo ] && sudo mkdir /sys/fs/cgroup/memory/demo  
echo "100000000" | sudo tee   /sys/fs/cgroup/memory/demo/memory.limit_in_bytes &>/dev/null  
echo $$ | sudo tee/sys/fs/cgroup/memory/demo/tasks &>/dev/null 
cat /dev/zero | head -c $1 | tail

$ ./memlimit.sh 82m  
```
參數解說:  
100000000單位bytes，接近100m  
-c: 一次顯示一個字元  
/dev/zreo : 一直丟零出來  
tail: 接收到enter[13]才會結束   
82m: 代表在跑這一支程式時會被霸占大約82m的記憶體  

2. 控制群組使用cpu的資源   
 \$ cat /sys/fs/cgroup/cpu/cpu.shares  
 (看cpu的大小，預設為1024)  
\$ sudo mkdir /sys/fs/cgroup/cpu/low  
\$ sudo mkdir /sys/fs/cgroup/cpu/high  
(創建兩個群組low,high)  
\$ echo 512 | sudo tee /sys/fs/cgroup/cpu/low/cpu.shares  
\$ echo 2048 | sudo tee /sys/fs/cgroup/cpu/high/cpu.shares  
(給low,high的比率，此設定為1:4)  
\$sudo chown bigred:root -R /sys/fs/cgroup/cpu/low  
\$sudo chown bigred:root -R /sys/fs/cgroup/cpu/high    
(-R=recursive，設定兩個群組的使用者皆為bigred)   

3.  cpuset控制群組  
\$ cat /sys/fs/cgroup/cpuset/cpuset.cpus  
0-1 (檢視原本 cpuset 裡的設定，0-1表示有兩核)  
\$ sudo mkdir /sys/fs/cgroup/cpuset/first  
\$ sudo chown bigred:root -R /sys/fs/cgroup/cpuset/first  
(建立 cpuset 控制子群組 - first，並設定權限)  
\$ echo 0 | sudo tee /sys/fs/cgroup/cpuset/first/cpuset.cpus  
(設定 first 控制子群組, 將 first 控制子群組設成第一個 cpu core)  
\$ echo 0 | sudo tee /sys/fs/cgroup/cpuset/first/cpuset.mems  
\$ echo $PID | sudo tee /sys/fs/cgroup/cpuset/first/tasks  
(將測試程式的 PID 加入到 first 控制子群組)    
:arrow_right_hook:  
$ echo $PID | sudo tee /sys/fs/cgroup/cpu/low/tasks  
$ echo $PID | sudo tee /sys/fs/cgroup/cpu/high/tasks  
(將程式丟到low或high群組)

**$ top 指令可以查看 cpu 使用情況**  

---

## slirp4netns (Network namespace)  
**建立虛擬網路, 連接 Internet**   

:warning:   (eth為中華電信的高級網路處理器)  
Container 撥給slirp4(中華電信)再透過internet才可以上網
![slirp4](https://i.imgur.com/D73faI5.png)

替這個container創造專屬網路運作空間, 但還沒有網路
```
$ sudo unshare --pid --fork --mount-proc --net --uts sh
root@ctn:/home/bigred$ ifconfig -a
lo: flags=8<LOOPBACK>  mtu 65536
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

:arrow_right: **創造網路步驟**  
\$ sudo apk add slirp4netns   
\$ sudo modprobe tun  
\$ sudo slirp4netns --configure  --disable-host-loopback --mtu=65520 \$PID tap0      
(撥號給中華電信，中華電信自行處理後撥回網路給container)   
(然後他將開著持續接收中華電信的網路回撥)    
($PID 為執行 sudo unshare... 的 pid)  

---

## :pencil2: 實作Docker   
```
$ sudo apk add docker
```
:warning: 下載之後重要三步驟  
```
$ sudo rc-update add docker boot  
$ sudo addgroup bigred docker (將 bigred 帳號加入 docker 群組後，就不需使用 sudo 命令執行 docker)
$ sudo reboot
```
:earth_africa: **Docker 指令集**  
\$ docker info  
(查看套件資訊)  
```
.........
 Images: 0
 Server Version: 20.10.6 (年,月,修改次數)
 Storage Driver: overlay2
.......
```

\$ docker run  
\( 產生 container )  
參數 :  
\-d : 背景執行  
\-h : Docker的電腦名稱  
\-m : 限制記憶體  
\-f : 在前景執行(在指令裡面打的)  
\-it : 產生終端機(有貝殼程式就一定要有)  
\- -net : 設定使用的網型態  
\- -rm : container 結束執行的指令後自動被刪除  
\- -name : 給 container 的名字   
\- -cpus : 設定只能使用幾 % 的 cpu  
\- -cpuset-cpus : 設定使用的 core  
**- -privileged : 最高級指令**  
**- -restart=always : 隨主機重新開機而跟著啟動，因 Docker Host 重啟內定不會自動啟動 Container**  
  

```
$ docker run --rm --name c1 --cpuset-cpus="1" --cpus="0.2"  -itd  busybox  yes
```

\$ docker ps -a   
(查看所有 container 狀況)  

\$ docker start cname  
(啟動 container )    

\$ docker stop cname  
(停止 container )  

\$ docker rm cname  
(刪除 container ，-f 為強制刪除)  

```
$ docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                     PORTS               NAMES
54ab5a1a4690        busybox             "/bin/sh"           5 minutes ago       Exited (0) 9 seconds ago                       b1

$ docker start b1 (只有開機)
$ docker exec -it b1 sh (用2號使用者登入)

/ # ps
PID   USER     TIME  COMMAND
    1 root      0:00 /bin/sh
    7 root      0:00 sh
   15 root      0:00 ps
/ # exit (2號使用者離開, 但這台container並沒有被關機喔)

$ docker stop b1 (Stop關機沒有像開機那麼快是因為要通知裡面的application container正常關機，不是kernel關機喔!)
$ docker rm b1
```

\$ docker rmi imgname  
(刪除 image 檔)  

\$ docker stats cname  
(查看 container 檔使用資源)  

\$ docker update -m 896m m1  
(可動態改變 Container 記憶體限制)  

\$ docker exec   
(對自己存在的 container 下達指令)  

\$ docker network create 網路名  
(創造一個網路段)  

:mag_right: 
問題討論
```
(一) : 如何在為何以下命令執行錯誤 ?
$ docker run --rm --name=b1 -i -t dafu/busybox
FATA[0000] Error response from daemon: No command specified

(二) : 為何以下命令執行錯誤 ?
$ docker run --rm --name=b2 -i -t dafu/busybox  /bin/bash
FATA[0000] Error response from daemon: Cannot start container 1b1c5433af380fa2e5abb0c6b5b0b37ebb597573286e1db5e05b7cb9ec17d3fb: exec: "/bin/bash": stat /bin/bash: no such file or directory

(三) : 為何 Container 系統中不能修改日期與時間 ?
$ docker run --rm --name=b3 -it busybox
/ # date --set "2006-10-10 18:00:00"
date: can't set date: Operation not permitted
Tue Oct 10 18:00:00 UTC 2006
```
(一)執行container要process，但後面沒有說是要用sh/bash…或是其他prosess  
(二)拿到dafu/busybox的光碟片中沒有bash這個program, 所以出現錯誤資訊  
(三)**改的時間是改alpine的時間, 不是改container的時間!**  
原先的root capability權利被拔掉, 所以在這個container中無法更改日期，所以用privileged把這台container可以用的capability全部套用進去  
\$ docker run --rm --privileged --name=b3 -it busybox  

---

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

```
重新建立 testweb 貨櫃主機
$ docker run --name testa1 -d <login name>/a1 httpd -f -p 8888 -h www

$ docker exec -it testa1 hostname -i
172.17.0.2

$ curl http://172.17.0.2:8888
<h1>Busybox HTTPd</h1>

以強制方式移除 testa1
$ docker rm -f testa1
```
參數說明:  
-f: container在背景，container process一定要在前景執行第一支程式  
查IP: docker exec -it container名字 hostname -i  


```
$ docker run --rm --name n1 -p 8080:80 -d nginx

$ docker ps 
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                  NAMES
ad82bd0803fe        nginx               "nginx -g 'daemon of…"   47 seconds ago      Up 46 seconds       0.0.0.0:8080->80/tcp   n1

$ curl http://localhost:8080 
.........
<p><em>Thank you for using nginx.</em></p>
</body>
</html>

可動態改變 Container 記憶體限制
$ docker update -m 896m m1

$ docker stats m1
CONTAINER ID   NAME      CPU %     MEM USAGE / LIMIT   MEM %     NET I/O     BLOCK I/O        PIDS
966e877eeb07   m1        0.00%     2.895MiB / 896MiB   0.32%     876B / 0B   573kB / 8.19kB   2
^C

$ docker stop m1
```

:warning:   
8080是在Alpine上面開的port號，所以在連網站的時候localhost = alpine的IP ; 80是開在裡面的container上的port號  
所以可以用windows連alpine IP，**但若網站被攻陷是container受傷，因為alpine就像是router，實際連到的是container**

---

### :bridge_at_night: Docker橋接網路  

\$ docker  network  ls  
(查看有什麼網路類型)  

基本三種 **(其中Bridge與none的模式都不會與上一層有掛鉤, 但host mode會使用到上一層真正的資源)**:

1. **Bridge mode** (=VMware 的NAT，使用虛擬橋接器)
based on上一層主機資源，在這個環境中的container若要使用priviliged指令強制刪除apline網卡雖然在container中執行，但exit出來之後alpine的網卡是不會受到影響的  

2. **Host mode** (不是VMware 的hostonly，使用主機所有網路介面，此模式不建議使用)  
直接使用到上一層主機資源, 在這個環境中的container若要使**用priviliged指令強制刪除apline網卡, alpine的網卡是真的會被刪除掉的**, 所以這個環境是盡量不要去做使用, 容易有資安上面的疑慮

3. None (沒有網路介面，但可以手動自製)  
**此模式中的container沒有網路卡但彈性最高, 可以自建網卡或是使用alpine的網卡資源**, 在這個環境中的container若要使用priviliged指令強制刪除apline網卡雖然在container中執行, 但exit出來之後alpine的網卡是不會受到影響的

\$ brctl show  
(看虛擬橋接器 Docker0 接了哪些網卡)  
```
$brctl show
bridge name	bridge id		STP enabled	interfaces
docker0		8000.02429b11cfa0	no		veth4d8bf88
........

$ ifconfig veth4d8bf88
veth4d8bf88 Link encap:Ethernet  HWaddr 3e:69:19:3e:c9:fe  
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
                                              :::
```
 ### 新增 Docker 網路
```
$ docker network create mynet1

顯示 mynet1 的 Network ID
$ docker network inspect --format='{{range .IPAM.Config}}{{.Subnet}}{{end}}'  mynet1
172.18.0.0/16

$ docker network create --driver=bridge --subnet=192.168.166.0/24 --gateway=192.168.166.254  mynet2
436f883e897378596577d3d4cf3efd691f0c0f7159060db2905373e14def8042
$ docker network ls 
NETWORK ID       NAME          DRIVER
..........
1eecce80201c     mynet1        bridge
436f883e8973     mynet2        bridge

$ sudo iptables -t nat -L -n | grep MASQUERADE
MASQUERADE all -- 192.168.166.0/24   0.0.0.0/0
MASQUERADE all -- 172.18.0.0/16   0.0.0.0/0
MASQUERADE all -- 172.17.0.0/16   0.0.0.0/0

```

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
參數說明:  
-b: 創造網路卡  
-u: 給專屬的使用者  

:+1: **撰寫 dknet 程式:**  
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

*出自陳松林老師
```
### :cloud: Ubuntu Docker用實體網卡  
```
$ docker run --rm --name d1 --cap-add=NET_ADMIN --net='none'  -itd busybox  sh
3879c04ac6435e5cd0a64effdc5843bce57c3b1e4f277722806f622af53d4425

確認好alpine是使用哪張網卡(ifconfig -a)之後,將該實體網卡加入 d1 貨櫃主機的Network Namespace
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
```

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

## :hamster: Dockerfile   
**Dockerfile 就是建置 Docker Image 的腳本**
### :pencil: 撰寫 Dockerfile  
1. 先建造 Docker file 所在的目錄區  
```
$ mkdir -p ~/wulin/myring; cd ~/wulin/myring
```
2. 開始撰寫 dockerfile  
```
$ echo 'FROM alpine
RUN echo "top.secret" > /password.txt
RUN rm /password.txt' > Dockerfile

docker命令:  
FROM alpine (從官網下載光碟片做成我的光碟片)  
RUN 在 container 中執行 linux 命令  
```
3. 建立與測試 Docker image   
```
$ docker build --no-cache -t myring .
$ docker images
REPOSITORY   TAG       IMAGE ID          CREATED              SIZE
myring           latest    97f72354b5a6    9 seconds ago      5.61MB
alpine             latest    6dbb9cc54074   3 weeks ago         5.61MB

$ docker run --rm myring  ls /password.txt
ls: /password.txt: No such file or directory
前面已經有刪除指令，但是password.txt還在!
```
參數說明:  
no-cache : 從(官網)網路上下載完不會做暫存，若不加此指令則機器會存有舊的光碟片，日後光碟片有更新，機器還是會用舊的。  
\-t : 命名  
\. : 在當前目錄找 Dickerfile  


---

## Docker image 目錄結構  
![dir](https://i.imgur.com/WsUCu24.png)  

**架構解說 :**   
Docker file 的內容，會有很多資料夾(一個指令產生一個)  
FROM : 產生container  
Java 開發平台的相依檔 : openjdk版本8，這樣在alpine就可以使用java application  
ARG : 宣告變數，下參數  
COPY : 從 linux 中 copy 到 CD 中  
\.jar : java application  
ECTRYPOINT 啟動光碟片 : “java”平台 “-java”參數 “/app.jar”應用程式  
final docker image: lower資料夾可以掛載多個資料夾, 從Docker image來  

*今天在運作 container 的時候會，有一個空資料夾(merged)讓 overlay2 可以 mount 進去 Image (資料夾)會堆疊在 overlay2的 Lower (可以看到所有的檔案)；Upper 給 container 使用(R+W)  
Container 的檔案系統為被 overlay2 覆蓋的 merged  

### 研究 Docker image  
```
$ docker save myring > myring.tar

$ mkdir imglayers; tar -xf myring.tar -C imglayers/

$ tree imglayers/
imglayers/
├── 579fb4fe2cc21a56bb5518ce0e55a8150ed388221deefb09657948ec5d3bfd79
│   ├── VERSION
│   ├── json
│   └── layer.tar
├── 824341e8ce2bb06b616b9116547f7fff6df15d72b9a9465e20b97e055e2747ec
│   ├── VERSION
│   ├── json
│   └── layer.tar
├── 97f72354b5a6c180f16ebf83b43d4ba876c0c70ae99bde2a06a1fefaaafb8cf4.json
├── cb6b5a6bbd1d00cdb311f08b7c4e4ed6320eac032177c6722072d1371b0c63f0
│   ├── VERSION
│   ├── json
│   └── layer.tar
├── manifest.json
└── repositories

$ cat imglayers/manifest.json | jq
[
  {
    "Config": "97f72354b5a6c180f16ebf83b43d4ba876c0c70ae99bde2a06a1fefaaafb8cf4.json",
    "RepoTags": [
      "myring:latest"
    ],
    "Layers": [
      "579fb4fe2cc21a56bb5518ce0e55a8150ed388221deefb09657948ec5d3bfd79/layer.tar",
      "824341e8ce2bb06b616b9116547f7fff6df15d72b9a9465e20b97e055e2747ec/layer.tar",
      "cb6b5a6bbd1d00cdb311f08b7c4e4ed6320eac032177c6722072d1371b0c63f0/layer.tar"
    ]
  }
]
(沒jq套件的話，記得要載)
(Json檔會記錄dockerfile all command)

$ cat imglayers/599588866dbf2a29d2a3a39587f6d65994c86566af280150dfe09767cba41389.json | jq
{
  .......
  "history": [
    {
      "created": "2021-04-14T19:19:39.267885491Z",
      "created_by": "/bin/sh -c #(nop) ADD file:8ec69d882e7f29f0652d537557160e638168550f738d0d49f90a7ef96bf31787 in / "
    },
    {
      "created": "2021-04-14T19:19:39.643236135Z",
      "created_by": "/bin/sh -c #(nop)  CMD [\"/bin/sh\"]",
      "empty_layer": true
    },
    {
      "created": "2021-05-11T15:50:10.495312977Z",
      "created_by": "/bin/sh -c echo \"top.secret\" > /password.txt"
    },
    {
      "created": "2021-05-11T15:50:11.067783811Z",
      "created_by": "/bin/sh -c rm /password.txt"
    }
  ],
.........
```
### 檢視 Docker image Layer 0 & 1 & 2 內容
```
$ mkdir rootfs; tar -xf imglayers/579fb4fe2cc21a56bb5518ce0e55a8150ed388221deefb09657948ec5d3bfd79/layer.tar -C rootfs

$ tree -L 1 rootfs
rootfs
├── bin
├── dev
├── etc
├── home
├── lib
├── media
├── mnt
├── opt
├── proc
├── root
├── run
├── sbin
├── srv
├── sys
├── tmp
├── usr
└── var

$ tar -xf imglayers/763a169d6c4ed9a83f2ef9f0b0b79b112303a53c4f778fc78261570eeab9f32f/layer.tar -C rootfs

$ ls -al rootfs/password.txt
-rw-r--r-- 1 bigred bigred 11 May 11 23:50 rootfs/password.txt
(前面已經有刪除指令,但是password.txt還在!)
$ tar -xf imglayers/c724281fbd49291c1338e29e87fb2cb514dfb8e97aabf4f351819ea2683ab2c6/layer.tar -C rootfs

$ find rootfs -name '*.txt'
rootfs/password.txt
rootfs/.wh.password.txt
(.wh.=white out 立可白檔,隱藏後方的檔案(password.txt))
```
:+1: 如果改成以下寫法，則在 docker image 裡不會產生資料夾，換句話說，docker image 不會產生空白的資料夾。  
```
$ echo 'FROM alpine
RUN echo “top.secret” > /password.txt && rm /password.txt
```
---

## 開發 Golang 網站  
```
$ cd ~/wulin; mkdir mygo; cd mygo

$ echo 'package main
import (
	"fmt"
	"log"
	"net/http"
)
func handler(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintln(w, "Hello, 世界")
}
func main() {
	http.HandleFunc("/", handler)
	log.Fatal(http.ListenAndServe(":8888", nil))
}' > main.go

$ go mod init mygo
(啟動相依檔, 把 Golang 當作第一支執行的程式)
$ CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -o main
```
架構說明 :  
package main 程式的相依檔:  
fmt: format 顯示格式的套件  
log: 記錄所有命令  
net: 要可以執行httpd  
func main : 從此處開始執行  
mod = module  

### 自製原生 Docker Image  
```
$ dir
total 5.9M
drwxr-sr-x 2 bigred bigred 4.0K Apr  9 00:12 .
drwxr-sr-x 3 bigred bigred 4.0K Apr  9 00:09 ..
-rw-r--r-- 1 bigred bigred   21 Apr  9 00:09 go.mod
-rwxr-xr-x 1 bigred bigred 5.9M Apr  9 00:12 main
-rw-r--r-- 1 bigred bigred  234 Apr  9 00:09 main.go

$ echo 'FROM scratch
ADD main /
CMD ["/main"] ' > Dockerfile

$ docker build -t goweb  .
(-t : 命名)
$ docker images
REPOSITORY    TAG         IMAGE ID            CREATED             SIZE
goweb             latest       7f9652539fc0      14 minutes ago    6.13MB
```
架構說明 :  
main : 翻譯好的 go lang application(含有相依檔，自走砲)，可以將這個檔案SCP到其他台電腦，就可以在別台使用了  
scratch : 空白光碟片  
ADD : 把已經有的檔案 copy 到指定的目錄  
CMD : 只要沒給命令內定就跑 go application  
(如果要推光碟小舖，記得光碟名要取 : 小舖名稱ㄥ)



