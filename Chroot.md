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