# Chroot (4/29)  
舊名sandbox(沙箱)  

1.  mkdir rootfs; cd rootfs; curl -s -o alpine.tar.gz http://dl-cdn.alpinelinux.org/alpine/v3.13/releases/x86_64/alpine-minirootfs-3.13.4-x86_64.tar.gz;  
:arrow_right:   
創造一個資料夾 rootfs，然後在裡面下載 alpine 根目錄下的所有檔案  
2.  tar xvf alpine.tar.gz &>/dev/null ;rm alpine.tar.gz; cd ..   
:arrow_right:  
xvf : 解壓縮參數(記得刪掉壓縮檔，才不會一直堆積東西)  
3.  sudo chroot rootfs/ /bin/sh  
:arrow_right:  
把 rootfs 更改為在根目錄下執行，且使用的是 /bin/sh 貝殼程式(進去後可使用 \$whoami 檢查)  


---
## 指定要更改的使用者(不要root)
1.  sudo chroot --userspec=bigred:bigred  rootfs/ /bin/sh  
:arrow_right:  
--userspec : 指定使用者的參數  
:warning:  **進去後會沒有使用者，因為是在 rootfs 資料夾裡原始的 alpine**  
:gemini: *所以記得要先進去使用 root 權限創造你要的使用者*  


---


---

# Cgroup(4/29)
分配資源(cpu,core,memory)
1. memory控制群組  
\$ sudo mkdir /sys/fs/cgroup/memory/demo  
(在cgroup之下創建的目錄會繼承上一層目錄的資料)  
:i_love_you_hand_sign: **(重要)**  
\$ nano memlimit.sh   
#!/bin/bash  
[ ! -d /sys/fs/cgroup/memory/demo ] && sudo mkdir /sys/fs/cgroup/memory/demo  
echo "100000000" | sudo tee   /sys/fs/cgroup/memory/demo/memory.limit_in_bytes &>/dev/null  
echo $$ | sudo tee/sys/fs/cgroup/memory/demo/tasks &>/dev/null    
cat /dev/zero | head -c $1 | tail    
:arrow_right: *(單位bytes，接近100m)(-c:一次顯示一個字元)(/dev/zreo : 一直丟零出來)(tail:接收到enter[13]才會結束)*  

2.  cpu控制群組  
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
(設定兩個群組的使用者皆為bigred)   



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
:a: 使用 $ top 指令可以查看 cpu 使用情況  


---



  

