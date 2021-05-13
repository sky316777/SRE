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
:earth_africa: Docker 指令集  
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




:cd: 關於光碟的指令  

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

                  



