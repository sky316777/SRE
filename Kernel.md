# Kernel  

每個電腦裡面都有一個operating system, 
kernel是linux作業系統中的核心程式, 可以管理、控制硬體ex. Mem、HD、cpu、Network。  
kernel空間的module是動態的, 需要使用時才會掛載進來。  
要使用kernel space必須要有root的權限

*operating system 電腦作業系統  
ex. Ubuntu Server / Centos / Apline  

![;kernel](https://i.imgur.com/w4GpEfg.png)


**查詢Linux中有多少module的指令:  
$ lsmod**

---

# Linux 的開機流程  

### step 1 : Power on  

### step 2 : BIOS/UEFI  

* 檢查硬體  
檢查I/O設備(POST)，無誤就發出"逼"一聲  
* 讀取第一個可開機裝置的MBR  
來源可能是硬碟、光碟、USB  

### step 3 : Boot loader  

* 放在第一個硬碟的sector(磁區)，啟動作業系統  
boot loader種類  

| 種類      |                      |
| -------- | -------------------- |
| GRUB2    | 以硬碟啟動OS (kernel) |
| ISOLINUX | 以光碟啟動OS (kernel) |
| SYSLINUX |  可以硬碟也可以光碟     |

### step 4 : Kernel

* 啟動Kernel  
會得知I/O資訊，將硬體資訊 (cpu/ram/hd/網卡) 記錄到在Ram形成的檔案系統目錄:  
**/proc、/dev**

### step 5 : Init  

* kernel將控制權交給第一個程式
* 各LinuxOS的init



| Linux OS | first process |
| -------- | --------      |
| Ubuntu     |Systemd      |
| Alpine     |    init     |
|TinyCore    |busybox      |
| Centos     | Systemd     |

* init再啟動以下程式  
Daemon背景程式: sshd、httpd、dhcp、squid  
login (帳密) 前景程式	  

---

## 登入系統之後:  
* user/passwd
* **/etc/profile (開機後第一個執行的程式)**  
可以放各種自訂指令(alias、自動登入&秀ip...)，
這個檔案跟.bashrc是不一樣的!  
\$ sudo nano /etc/profile  
(之後用這個電腦備份，所有使用者都可以用)  
\$ sudo nano .bashrc  
(只 for 當前使用者這個帳號可以用 ex. bigred)  
* /bin/bash or /bin/sh (貝殼程式)  
* **$** (工作環境)  
1. home directory  
2. env  
\$ USER / $ IP / $ PATH  
**/bin/bash的原生命令:cd、exit、alias**  
*若把 $PATH 的路徑都清空, 則會導致程式無法執行 
```
$ echo $PATH
$ export PATH=
則代表路徑被清空
$ exit
重新登入即可
```
( A ) File system  
a. permission  
r ->ls、cat、head、tail  
w ->mkdir、touch,rm,rm -r  
x ->cd、~~source~~ (source不須給予執行權限也可引用裡面的內容)

b. setuid/setgid/sticky  
<1> setuid ->對檔案影響  
執行時以owner權限執行  
<2> setgid ->檔案及目錄  
檔案 ->是以擁有該文件的"群組"運行  
目錄 ->讓該目錄內的檔案不屬於創建者的群組，而是父目錄所屬的群組  
<3> sticky ->對目錄影響  
該目錄內所有檔案都只能被它的owner移動、刪除，但其他使用者有寫的權限  

c. umask  
檔案666  
目錄777  
預設022(就是拿掉group和other的w權限)  

( B ) Process  
/usr/bin/passwd  


( C ) Network  
a. IP (private IP)  
b. gateway 上網的出口  
c. routing table (route -n)  
d. iptables  
e. DNS  

