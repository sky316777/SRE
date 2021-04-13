# Alpine 安裝手冊

黃鉦翔

---
Alpine Linux

特色：
Alpine Linux
是一個由社群開發的基於 musl 和BusyBox 的 Linux 作業系統。最主要特色小且輕便。


[光碟下載官網網址](https://alpinelinux.org/downloads//)
https://alpinelinux.org/downloads//

選STANDARD的 x86_64

![](https://i.imgur.com/yYkfExs.png)



---

開始安裝
第一步
開啟 VMware
create一個新虛擬機
記得選第三個(之後再設定硬體的選項)
![](https://i.imgur.com/oPAmjoU.png)



選Linux 3.x kernel 64-bit
![](https://i.imgur.com/XDxm0Ec.png)




設定電腦名稱還有放置的位置
![](https://i.imgur.com/oJ1830x.png)




設定容量

***下面記得要選 Single file***
![](https://i.imgur.com/7SPe2hm.png)





按Customize Hardware 去設定模擬機
![](https://i.imgur.com/jq31S5u.png)




左邊沒用的可以刪掉(USB,Sound Card,Printer)

***記得要選 NAT***

***記得要放入光碟***

![](https://i.imgur.com/dKlOCqq.png)




完成後把他打開
![](https://i.imgur.com/EnAQf3R.png)




---




登入打 root
![](https://i.imgur.com/RcymkZF.png)



第一步先輸入 setup-alpine
![](https://i.imgur.com/3W2uI2R.png)


輸入國家和語言
連輸入兩個 tw
![](https://i.imgur.com/BwbG6Sf.png)


輸入電腦名稱 bigred 之類的
![](https://i.imgur.com/xzsoM9c.png)

之後設定網卡選項等等
簡單來說連按三次 Enter
![](https://i.imgur.com/0gT4xg9.png)

設定密碼 bigred 之類的
![](https://i.imgur.com/la9Fyz5.png)

設定時區
輸入 Asia
輸入 Taipei
![](https://i.imgur.com/JYXtekH.png)

設定一些有的沒的，預設的話按 Enter就好(兩次)
![](https://i.imgur.com/zIPnEHb.png)

有很多選擇給你下載，建議選37(沒有為什麼)
輸入 37 按 Enter
![](https://i.imgur.com/a41gIY6.png)

再來設定Openssh，按 Enter 即可
![](https://i.imgur.com/Hfule2O.png)

接下來很重要
輸入 sda 
輸入 sys
輸入 y
(有三張圖給你看過程)
![](https://i.imgur.com/9ab7sZN.png)
![](https://i.imgur.com/8hCRPeT.png)
![](https://i.imgur.com/Zf2Z1vy.png)

在下載了，請等待(會需要一段時間)
![](https://i.imgur.com/uv1jxOP.png)


下載好了，請重開機
***重開後一定要退出光碟***
![](https://i.imgur.com/A3fXPc4.png)
![](https://i.imgur.com/80wYTxq.png)


---

