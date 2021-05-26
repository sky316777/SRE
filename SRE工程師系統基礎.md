# SRE工程師系統基礎
### 1.認識程序與程式
program: 檔案需具有執行權 (chmod +x)
process: 泛指使用者在作業系統(Windows/MAC OS/Linux/AIX…)跑的軟體/程式(busybox網站…)

user把可以執行的program從硬碟載入至記憶體, 進入process之後會記錄啟用這個程式的人IP

![p1](https://i.imgur.com/22dy8xn.jpg)

-----

### 2.顯示程序資訊
( a ) 條列式顯示所有程序
`ps -eo user,pid,cmd,%mem,%cpu`

![p1](https://i.imgur.com/vn9Y7q1.png)

因為Alpine是一個比較精簡的軟體, 裡面的功能不見得全部都有, 所以要另外下載部分指令
 
![p2](https://i.imgur.com/IxBEhhv.png)

*busybox 多用在IOT裝置上因為程式檔很小, 可以執行linux很多指令。但小也有缺點, 因為它可以模擬的指令太多, 可能無法每個命令都全功能上線
![p3](https://i.imgur.com/ZL5OxcW.png)

![p4](https://i.imgur.com/H5sEe9f.png)

( b ) top批次顯示
如果今天電腦使用變慢, 下top可以看出當下佔使用量最多的process, 現行使用狀態
```
top  -bin  2  -d 1  >pid.txt
cat pid.txt
```
![p5](https://i.imgur.com/9Q5x2zL.png)

( c ) pstree
`pstree –ph(alpine-p就好/ubtune要-ph)`

有bash因為是透過powershell遠端連線, 若再開第二個powershell如下

**Alpine: Init: 開機後叫做第一個執行的程式
Ubuntu: Systemd: 開機後叫做第一個執行的程式**

![p6](https://i.imgur.com/5IClU8g.png)

![p7](https://i.imgur.com/gxsK17A.png)

### 3.子程序與父程序
```
echo $$ 查我自己程式的PID
$PPID 查父程式的PID
```

### 4.程序與環境變數

程式變數無法讓其他程式繼續繼承
若用export則其他程式就可以繼承此變數

### 5.程序與信號
### 6.背景執行程序

Ctrl + c 是中斷/ Ctrl + z 是暫停,背景還在執行
kill -2 PID/ kill -9 PID

### 7.程序優先順序

daemon: process在背景跑, 優先順序高 ex. Open ssh/ 資料庫/網站系統...
nice=友好程度越高優先順序越低
```
第一個參數代表運算的時間, 第二個參數代表名字
nano count1.sh 
#!/bin/bash

x="$1"
echo "$2" $(date)
while [ $x -gt 0 ]; do
    x=$(( x-1 ))
done
echo "$2" $(date) 

$ chmod +x count1.sh
```

![p8](https://i.imgur.com/V27G8lN.png)


-----------

## Process Security in Linux
### 1.prcess的防火牆系統: Seccomp
**Linux System Call & Seccomp (security computing mode)**
在linux執行中程式的防火牆

### 2.setuid/setgid/sticky

**setuid=4, 
setgid=2, 
sticky=1**


#### ( A ) setuid
當一個可執行文件啟動時, 不會以啟動它的用戶的權限運行, 而是以該文件所有者的權限運行 >>掠奪使用者權限  
所以，如果在一個可執行文件上設置了setuid，並且該文件由root擁有，當一個普通用戶啟動它時，它將以root權限執行。顯然，如果setuid使用不當的話，會帶來潛在的安全風險。  

ex. desktop建立一個檔案, 但若把某指令的執行權限setuid, 之後用該指令權限執行的程式owner會更改。
![uid](https://i.imgur.com/1RqpUyh.jpg)


#### ( B ) setgid
對文件和目錄都有影響, 只要其他使用者備加入主要owner的群組, 他就可以在群組內建立/刪除檔案與目錄

```
esktop@UB2004D:~$ sudo useradd -m -s /bin/sh joe

desktop@UB2004D:~$ sudo passwd joe

desktop@UB2004D:~$ ls -al | grep test

drwxrwxr-x  2 desktop desktop  4096  四  28 18:13 test

desktop@UB2004D:~$ sudo login joe

$ cd /home/desktop/test

$ nano 1
```
就會發現無法在這個目錄中新增內容
![](https://i.imgur.com/PyenX1b.png)

但desktop若更新這一個特定目錄/檔案的執行權限setgid, 並把joe加入至desktop的群組中, 他就可以在這一個目錄中新增/刪除任何資訊

```
esktop@UB2004D:~$ chmod 2775 test

desktop@UB2004D:~$ ls -al | grep test

drwxrwsr-x  2 desktop desktop  4096  四  28 18:13 test

esktop@UB2004D:~$ sudo addgroup joe desktop
Adding user joe to group desktop ...

Adding user joe to group desktop

Done.
```


#### ( C ) sticky
只有owner可以刪除/移動這個指定目錄下的檔案, 其他使用者只有修改內容的權限

```
desktop@UB2004D:~$ chmod 1777 test

desktop@UB2004D:~$ ls -al | grep test

drwxrwsrwt  2 desktop desktop  4096  四  28 18:41 test

desktop@UB2004D:~$ cd test

desktop@UB2004D:~/test$ touch 123

desktop@UB2004D:~/test$ sudo login joe

$ cd /home/desktop/test

$ ls -al

-rw-rw-r--  1 root    desktop    0  四  28 18:40 123

$ rm 123

rm: cannot remove '123': Operation not permitted
```


### 3.capabilities

列出有設定 Linux capabilities 的所有命令
`getcap -r / 2>/dev/null`

從根目錄開始搜尋只要沒有capabilities的目錄就導入到黑洞, 剩下的就是具有cap特別功能的目錄


-----------

### Linux 目錄與檔案之權限意義

#### 1.權限對檔案的重要性
#### 2.權限對目錄的重要性
#### 3.使用者操作功能與權限
R: cat/tree/ls/head/tail/cp/mv

W: nano/touch/cp/mkdir/rm;rm -r
#### rm: 被刪除的檔案/目錄要有W權限

X: cd目錄/執行檔案

---

## Process Security in Linux  
### 1. process 的防火牆系統 : seccomp  
**在執行階段的 process 有防護機制(防火牆)**
```
(檢視是否啟用 SECCOMP)
$ cat /boot/config-$(uname -r) | grep CONFIG_SECCOMP
CONFIG_SECCOMP=y
CONFIG_SECCOMP_FILTER=y
```

```
$ mkdir syscall; cd syscall

(安裝 go 開發套件)
$ sudo apt install golang-go

(安裝 Golang 所需的 Seccomp 套件)
$ sudo apt install libseccomp-dev golang-github-seccomp-libseccomp-golang-dev

$ sudo mv /usr/share/gocode/src/github.com/ /usr/lib/go-1.13/src/
(鎖定版本，不用隨意更新版本，新版不支援舊版)
(Ubuntu 查版本代號: go version)
```

**Seccomp 實作範例**  
```
$ nano main.go 
package main

import (
  "fmt"
  "syscall"
  "os"
)

func main() {

    var syscalls = []string{
    "rt_sigaction", "mkdirat", "clone", "mmap", "readlinkat", "futex", "rt_sigprocmask",
    "mprotect", "write", "sigaltstack", "open", "read","close", "fstat", "munmap",
    "brk", "access", "getrlimit","exit_group","getpid"}

    whiteList(syscalls)

    err := syscall.Mkdir("/tmp/moo", 0755)
    if err != nil {
        panic(err)
    } else {
        fmt.Printf("I just created a file\n")
    }
    fmt.Printf("pid: %d\n", os.Getpid())
}

$ go build -o test
(當前目錄區把所有.go程式翻譯出來產生一個program = test)
$ ./test
.......
I just created a file
pid: 35701
```
:arrow_right: main.go 裡面是 go 語言，寫任何一個程式語言寫出來都有一個相依檔，java/golang叫做 package(套件包)  
nil : 空值


### 2. Linux File Permission & Setuid   
**setuid = 4** : 對目錄沒有影響。setuid 位是用 s 來表示的，代替了可執行位的 x。小寫的 s 意味著可執行位已經被設置，否則你會看到一個大寫的 S。  
大寫的 S 發生於當設置了 setuid 或 setgid 位、但沒有設置可執行位 x 時。它用於提醒用戶這個矛盾的設置：如果可執行位未設置，則 setuid 和 setgid 位均不起作用。  

**setgid = 2** : 對文件和目錄都有影響，通常用於文件共享。  

**sticky = 1** : 它對文件沒有影響，但當它在目錄上使用時，所述目錄中的所有文件只能由其所有者刪除或移動。  

**umask** : umask的用法與chmod相反，chmod是在「000」上面「增加」權限，而umask則是在「666」基礎上「減少」檔案權限; 以及在「777」基礎上「減少」目錄權限。  
例如:  
\$ umask 004 # 就是將others的r權限移除  
\$ umask 022 # 就是將group和others的w權限移除  
\$ umask 111 # 就是將owner, group和others的x權限移除  

```
$ echo  'package main
import (
    "fmt"
    "io/ioutil"
)
func main() {
    err := ioutil.WriteFile("/mulan.txt", []byte("Hello"), 0755)
    if err != nil {
        fmt.Printf("Unable to write file: %v\n", err)
    } else {
        fmt.Printf("/mulan.txt created\n")
    }
} ' > myfile.go

$ CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -a myfile.go
```

```
(bigred 帳號沒有權限在 根目錄 (/) 產生檔案)
$ dir myfile
-rwxrwxr-x 1 bigred bigred  2.0M  Jul 26 05:13 myfile

$ ./myfile
Unable to write file: open /mulan.txt: permission denied

(一定要先設定 owner, 才可設定 setuid)
$ sudo chown root myfile; sudo chmod 4755 myfile

$ dir myfile
-rwsr-xr-x 1 root bigred 2.0M Jul 26 06:57 myfile

$ ./myfile 
/mulan.txt created

$ dir /mulan.txt 
-rwxr-xr-x 1 root bigred 5 Jul 26 07:09 /mulan.txt
```
```
(檢視系統中有多少具有 setuid 功能的命令)
$ sudo find / -user root -perm -4000 2>/dev/null | grep -E '^/bin|^/usr/bin'
/bin/umount
/bin/su
/bin/fusermount
/bin/mount
/usr/bin/chsh
/usr/bin/pkexec
/usr/bin/sudo
/usr/bin/passwd
/usr/bin/chfn
/usr/bin/newgrp
/usr/bin/gpasswd
```

### 3. capabilities  
**設給program，掠奪root的權限**  
```
$ sudo useradd -m -s /bin/bash rbean
$ echo "rbean:rbean" | sudo chpasswd

(設定 rbean 家目錄中的 python3 命令檔，具有 Capabilities setuid 權限，代表 python3 所執行的程式可設定 setuid 功能)
$ sudo cp /usr/bin/python3 /home/rbean 

$ sudo setcap  cap_setuid+ep  /home/rbean/python3
(讓python3有setUID)
$ exit


(列出有設定 Linux capabilities 的所有命令，從根目錄開始掃一遍哪些有被設定 capabilities)
$ getcap -r / 2>/dev/null
/home/rbean/python3 = cap_setuid+ep
/bin/ping = cap_net_raw+ep
/usr/lib/x86_64-linux-gnu/gstreamer1.0/gstreamer-1.0/gst-ptp-helper = cap_net_bind_service,cap_net_admin+ep
/usr/bin/mtr-packet = cap_net_raw+ep
/usr/bin/traceroute6.iputils = cap_net_raw+ep


(由以下命令得知 python3 命令檔並沒有設定 setuid 功能)
$ ls -al python3
-rwxr-xr-x 1 root root 5453504 Jul 28 04:56 python3

在以下 python3 命令所執行的 程式可執行 os.setuid(0) 這行命令, 提升 /bin/bash 這命令為 root 權限
$ ./python3 -c 'import os;os.setuid(0);os.system("/bin/bash")'
root@ub204:~# id
uid=0(root) gid=1001(rbean) groups=1001(rbean)
```
參數解說:  
\-c 跑程式  
os.setuid(0) 設root  
os.system(“/bin/bash”) 啟動時他的UID是root的0  
