# SRE工程師系統技術  
## Linux 程序管理  


1. 認識程序與程式(process&program)  
程式被使用者執行後，被叫到記憶體且會被記錄pid，然後被執行的程式成為程序  
2. 顯示程序資訊  
ps -p : 顯示程序資訊  
top : 批次顯示程序資訊  
pstree : 以樹狀顯示程序資訊  
which : 看目前檔案在哪個目錄  
3. 子程序與父程序  
\$$ : 目前程式的PID      
\$PPID : 目前程式的父親PID    
4. 程序與環境變數  
Export & Source  
行規 : 環境變數一律大寫  
5. 程序與信號  
程序是可以接受訊號的(看本身程式檔的寫法)  
\$trap "執行?" 訊號  
範例 : \$trap "echo hello" SIGINT  
6. 背景執行程序   
像是Openssh，網站，資料庫等等，先執行且在背景執行的重要程式，統稱為:daemon  
按ctrl+z 或者在執行程式時加上&，可讓程式在背景執行  
7. 程序優先順序  
\$nice -n 改變程式優先順序  
範例 : \$nice -n 19 ./count1.sh 5000000 B&  


---

## Process Security in Linux
1. process 的防火牆系統 : seccomp  
2. setuid  
3. capabilities  



---
## Linux 目錄與檔案之權限意義
1. 權限對檔案的重要性
2. 權限對目錄的重要性
3. 使用者操作功能與權限

---





