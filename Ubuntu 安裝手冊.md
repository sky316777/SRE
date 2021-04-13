# Ubuntu Server 安裝手冊



---
先去下列網站下載光碟
[ubuntu server下載網址](https://ftp.ubuntu-tw.org/ubuntu-releases//)  
選最新版(20.04.2.0)  

![](https://i.imgur.com/kHx9KZb.png)


---
開啟 VMware 創造一個新的模擬機  
選第三個  

![](https://i.imgur.com/FzvjCfw.png)


---
選Linux，Ubuntu 64-bit  

![](https://i.imgur.com/AqmFI7P.png)


---
設定電腦名稱還有所放位置  

![](https://i.imgur.com/kgHWvbq.png)


---
設定容量  
***下面記得選第一個，Single file 不然你會很麻煩***  

![](https://i.imgur.com/XCOzqID.png)


---
按 Customzie Hardware 進去設定  

![](https://i.imgur.com/upqndBX.png)



---
左側沒用的可以刪掉  
***最重要的是要放光碟***  

![](https://i.imgur.com/GapCPvm.png)



---
完成後把他打開  

![](https://i.imgur.com/PhL19Ug.png)


---
設定Server語言環境選取English， 因為沒有中文。  

![](https://i.imgur.com/aiDvpdi.png)


---
設定鍵盤布局台灣使用的都是美式鍵盤，請選擇 English(US)  


![](https://i.imgur.com/q5GCsJu.png)


---
設定網路ens33 為 Ubuntu Server 的網卡名稱虛擬機  
IP位址為DHCPv4給的  

![](https://i.imgur.com/aweWn80.png)


---
設定 Proxy address  
Proxy address為公司內部使用者只允許透過一個地址(網路終端)去連網，是大單位、或是很保護資安的公司才會需要設定。  

![](https://i.imgur.com/0jbpyHR.png)


---
設定Mirror address  
Canonical公司跟各地公司合作，將Ubuntu 對外發行的版本，備份在合作公司伺服器上，以便就近的使用者使用。  

![](https://i.imgur.com/VWeIx67.png)


---
設定硬碟安裝  
硬碟設定  

![](https://i.imgur.com/4jvNFMx.png)


---
硬碟分割區保留區  
(依據每個人電腦規格不同，這裡也會不相同)  

![](https://i.imgur.com/k8zVj3q.png)



---
按 Continue  

![](https://i.imgur.com/eoo4EjP.png)


---
設定帳號密碼Username是帳號名稱並且Username只能輸入0到9跟小寫英文字母  

![](https://i.imgur.com/rJRBYat.png)
![](https://i.imgur.com/Zjt9LJz.png)


---
是否安裝OpenSSH，如果需要遠端連線就需要在此設定，這邊的打 叉代表確認安裝喔!  

![](https://i.imgur.com/tkJ4SvM.png)
![](https://i.imgur.com/titMwU9.png)


---
安裝Linux 套件  
此為snap安裝，它徹底解決Linux依賴性的問題，也更擁有更佳穩定和安全特性。依照每個人的個人需求安裝。  

![](https://i.imgur.com/dG5bgyD.png)


---
等待安裝OpenSSH  

![](https://i.imgur.com/bSzfzAq.png)


---
安裝完成，點選reboot  

![](https://i.imgur.com/YR1BF5R.png)



---
按Enter，即可登入  

![](https://i.imgur.com/0vLEEps.png)


---
## 重開機後，一定要記得退光碟  



