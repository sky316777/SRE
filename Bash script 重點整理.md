# Bash script 重點整理  


---
### 命令集  
https://blog.techbridge.cc/2017/12/23/linux-commnd-line-tutorial/  
https://www.pcnet.idv.tw/pcnet/linux/linux_command.htm  
http://linux.vbird.org/  


---
### 常用瀏覽資料的指令  

:christmas_tree: **tree**  
功能說明:以樹狀圖列出目錄的內容。  
參 數:  
 -a   顯示所有檔案和目錄。  
 -d   顯示目錄名稱而非內容。  
 -f   在每個檔案或目錄之前,顯示完整的相對路徑名稱。  
 -s   列出檔案或目錄大小。  
 -t   用檔案和目錄的更改時間排序。  
 -u   列出檔案或目錄的擁有者名稱,沒有對應的名稱時,則顯示使用者識別碼。  
 tree -L 1 -d 只顯示一級目錄  
 
**pwd**：print work directory，印出(顯示)目前工作目錄(類似dos之CD Enter)  

**dirs**: 查現行目錄(類似dos之CD Enter)  

**mkdir**: 創建目錄  
$ mkdir -p bigred/{bin,home/{sys1,sys2}.tmp}  
(-p：連續建立兩個以上不存在的目錄)

**touch**: 創建空檔案 

**nano** : 修改或創建檔案內容  

**rm**  -參數  檔案或目錄  
(-r：刪除其下的檔案及目錄)

**cp** : 複製目錄或檔案或是更改命名(原始資料還健在)  

**mv** : 移動目錄或檔案或是更改命名(原始資料已故)  

**ls**：list，查看檔案及子目錄  
-l 列出詳細資料   
-a 列出隱藏資料  
-h  human人性化  

**cat**：顯示檔案內容  

**zcat**：顯示壓縮檔案裡的內容

**head** : 用於顯示檔的開頭的內容。在預設情況下，head命令顯示檔的頭10行內容。  
-n<數字>：指定顯示頭部內容的行數 ( +3 : 看前三列 ; -3: 刪除後三列 )   
-c<字元數>：指定顯示頭部內容的字元數  
-v：總是顯示檔案名的頭資訊  
-q：不顯示檔案名的頭資訊 

**tail** : 用於輸入檔中的尾部內容。預設在螢幕上顯示指定檔的末尾10行。  
-n<數字>：指定顯示尾部內容的行數 ( +3 : 刪除頭兩列 ; 3: 顯示最後三列 )  

**fmt** : 會從指定的檔裡讀取內容，將其依照指定格式重新編排後，輸出到標準輸出設備  
-u : 每個單字用一空格隔開  

**sort** : 它將檔進行排序，並將排序結果標準輸出  
-n：依照數值的大小排序  
-r：以相反的順序來排序  

**uniq** : 用於報告或忽略文件中的重複行，一般與sort命令結合使用。

**wc**　: 計算檔的Byte數、字數或是列數，若不指定檔案名稱，則wc指令會從標準輸入裝置讀取資料   
-l : 只顯示列數(lines)  
-w : 只顯示字數(word)  
-c : 只顯示Bytes數  

**tr** : 可以對來自標準輸入的字元 進行替換、壓縮(squeeze)和刪除。它可以將一組字元變成另一組字元  
-d : 刪除資料單引號中的字元  
-s : 把連續重複的字元以單獨一個字元表示  
```
bigred@bean:~$ echo "abcd abcd aa bb cc" |tr "bdc" "243"  
a234 a234 aa 22 33  
bigred@bean:~$ echo "aa bb cc"|tr -s "abc"  
a b c  
```

**cut** : 逐行擷取部份字元或欄位資料  
-c list: 一段範圍清單, 以','隔開, 直列切割(ex: 1,3,5-10,33)  
-d delim: 可以設中間間隔符號要哪種, 預設是tab分隔切割, 通常會搭配 -f(決定要取切割欄位的哪欄)  
-f list: 以欄位為主, 作剪下的動作, list 是欄位編號或一段範圍的清單(類同 -c 參數; ex: 1,3,5-10,33)

**tee** : 將結果同時輸出到螢幕和檔案(同時做兩件事)    
```
sudo echo nameserver 8.8.8.8 > /etc/resolv.conf
儘管這個解釋聽起來很合理，但它卻行不通，
因為出現錯誤，您將得到：
bash: /etc/resolv.conf: Permission denied
為什麼？
輸出重定向由您的外殼程序（在這裡：Bash）處理，
而用於的命令未考慮在內sudo。
類似於數學中的乘法優先於加法。
因此，只是echo具有更高的特權，而您的shell卻沒有！
為了解決這個問題，應該運行以下命令
echo nameserver 8.8.8.8 | sudo tee /etc/resolv.conf 

```


**grep** : 檔案內找尋關鍵字    
-c <顯示列數>或-<顯示列數>  除了顯示符合範本樣式的那一列之外，並顯示該列之前後的內容。  
-d <進行動作> 當指定要查找的是目錄而非檔時，必須使用這項參數，否則grep命令將回報資訊並停止動作。  
-E 將範本樣式為延伸的普通標記法來使用，意味著使用能使用擴展規則運算式。    
-f<範本文件> 指定範本檔，其內容有一個或多個範本樣式，讓grep查找符合範本條件的檔內容，格式為每一列的範本樣式。
-w 只顯示全字符合的列。  
-x 只顯示全列符合的列。  
-o 只輸出檔中匹配到的部分。  

***利用正規表示法***   
使用^和$來強制正則表達式分別僅在行的開頭或結尾進行匹配  
[[:alnum:]] -字母數字字符。  
[[:alpha:]] –字母字符  
[[:blank:]] –空白字符：空格和製表符。  
[[:digit:]] –數字：“ 0 1 2 3 4 5 6 7 8 9”。  
[[:lower:]] –小寫字母：“ abcdefghijklmnopqrstu vwxy z”。  
[[:space:]] –空格字符：製表符，換行符，垂直製表符，換頁符，回車符和空格。  
[[:upper:]] –大寫字母：“ ABCDEFGHIJKLMNOPQRSTU VWXY Z”。  
[[：punct：] ]：所有標點字符 
```
bigred@bean:~$ cat txt02  
11AA bb # test  
mary & john  
Dear John  
o Who i hate to write  
5201314  
No three N0 four  
bigred@bean:~$ grep '[[:digit:]]' txt02   
11AA bb # test  
5201314   
No three N0 four  
```




**echo**: 在螢幕上列印出指定的字串  
echo  字串  
**雙引號可有可無，單引號主要用在原樣輸出字串** 
```
若有空白字元可以使用雙引號『 " 』或單引號『 ' 』來將變數內容結合起來。雙引號內保有變數特性
$ name=victor; myname="variable name is $name"  
$ echo $myname  
variable name is victor
```
echo  $變數  
ECHO  ${變數}  
echo 字串  > 檔案(覆蓋)    
echo 字串  >> 檔案(累加)  
```
~$ echo 1111 >t1
~$ cat t1
1111

~$ echo 2222 >t1
~$ cat t1
2222

~$ echo 3333 >> t1
~$ cat t1
2222
3333
```
**echo -e 處理特殊字元**  

若字串中出現以下字元，則特別加以處理，而不會將它當成一般文字輸出：  
\a 發出警告聲；(alert)  
\b 刪除前一個字元；(Backspace)  
\c 最後不加上分行符號號；  
\f 換行但游標仍舊停留在原來的位置；  
**\n 換行且游標移至行首；(New line)**  
\r 游標移至行首，但不換行；  
**\t 插入tab；**  
\v 與\f相同；  
\\\ **插入\字元(跳脫字元)**；  
\nnn 插入nnn（八進制）所代表的ASCII字元； 


```
範例
$ echo “\”It is a test\””  
結果將是：  
“It is a test”  
```
---
:dark_sunglasses: :dark_sunglasses: :dark_sunglasses: :dark_sunglasses: :dark_sunglasses: :dark_sunglasses: :dark_sunglasses: :dark_sunglasses: :dark_sunglasses: :dark_sunglasses: :dark_sunglasses: :dark_sunglasses: :dark_sunglasses: :dark_sunglasses: :dark_sunglasses: :dark_sunglasses: :dark_sunglasses: :dark_sunglasses: :dark_sunglasses: :dark_sunglasses::dark_sunglasses: :dark_sunglasses: :dark_sunglasses: :dark_sunglasses: :dark_sunglasses:  
![](https://i.imgur.com/yxnefLx.png)  
可讀取（r，Readable），用數字 4 表示  
可寫入（w，writable），用數字 2 表示  
可執行：（x，eXecute），用數字 1 表示 
**chmod : 改變檔案或目錄的權限**  
 :arrow_heading_down: 
 **r** : cat, tree, ls, head, tail, cp, mv
 **w** : nano, touch, cp, mkdir, rm, rm -r
 **x** : cd, ~~source~~  
:warning: 刪除之檔案或目錄一定要有w權限  


---
### 變數設定規則與使用
:warning: 
**等號兩邊不能直接接空白字元(=兩邊不能空一格)**   
**變數需要在其他子程序執行，則需要以 export 來使變數變成環境變數  
(行規 : 環境變數要大寫)**

:wave:  
在一串指令中，還需要藉由其他的指令提供的資訊，指令先執行：使用 quote 『 \` command\`』，符號 \` 是鍵盤上方的數字鍵 1 左邊那個按鍵，而不是單引號。 
```
bigred@GW:~$ uname -r  
4.15.0-37-generic  
bigred@GW:~$ cd /lib/modules/`uname -r`  
bigred@GW:/lib/modules/4.15.0-37-generic$  
```

#### :arrow_heading_down:  各種括號的使用

{}：針對字串做置換、擷取處理、正規化  

[] : 條件式處理  

() : 代表收納命令處理後的字串  

(( )) : 第二個括號代表運算處理  

---
### 判斷符號的重點

:arrow_heading_down: **常見參數**  

![](https://i.imgur.com/igKgVwU.png)
```
$ echo $0  
myscript  
$ echo $1  
opt1  
$ echo $#  
4  
$ echo $@  
opt1 opt2 opt3 op4  
```

:arrow_heading_down: **判斷括號的使用**  
[ -d 目錄 ] : 判斷目錄是否存在  
[ -f 檔案 ] : 判斷檔案是否存在

&& 代表邏輯的 AND 的意思 : 兩者都成立才成立，其中一個不成立就不用做了    
|| 代表邏輯的 or的意思 : 其中一項式子成立就成立

:arrow_heading_down: **（^） 轉換成大寫；（,）轉換成小寫**  
（^）：把變數中的第一個字元換成大寫  
（^^）：把變數中的所有小寫字母，全部替換為大寫。  
（,）：把變數中的第一個字元換成小寫  
（,,）：把變數中的所有大寫字母，全部替換為小寫。  
```
bigred@bean:~$ pw="125dFgS4"
bigred@bean:~$ pw1=${pw//[0-9]/}
bigred@bean:~$ pw2=${pw1,,}
bigred@bean:~$ echo $pw
125dFgS4
bigred@bean:~$ echo $pw1
dFgS
bigred@bean:~$ echo $pw2
dfgs

```
:arrow_heading_down: **變數轉換**  
\${變數/舊字串/新字串} : 若變數內容符合『舊字串』則『第一個舊字串會被新字串取代』  
\${變數//舊字串/新字串} : 若變數內容符合『舊字串』則『全部的舊字串會被新字串取代』  

```
bigred@bean:~$ pw="123asdQWE_@"
bigred@bean:~$ pw1=${pw//[a-z]/}
bigred@bean:~$ pw2=${pw//[!a-z]/}
bigred@bean:~$ pw3=${pw//[A-Z]/}
bigred@bean:~$ pw4=${pw//[!A-Z]/}
bigred@bean:~$ pw5=${pw//[!A-Z0-9]/}
bigred@bean:~$ pw6=${pw//[!A-Z0-9a-z]/}
bigred@bean:~$ echo $pw
123asdQWE_@
bigred@bean:~$ echo $pw1
123QWE_@
bigred@bean:~$ echo $pw2
asd
bigred@bean:~$ echo $pw3
123asd_@
bigred@bean:~$ echo $pw4
QWE
bigred@bean:~$ echo $pw5
123QWE
bigred@bean:~$ echo $pw6
123asdQWE

```
:arrow_heading_down: **變數的更改刪除**  
${變數#關鍵字} : 若變數內容從頭開始的資料符合『關鍵字』，則將符合的最短資料刪除    
${變數##關鍵字} : 若變數內容從頭開始的資料符合『關鍵字』，則將符合的最長資料刪除  
```
$ aa="aaa:bbb:ccc:ddd:aaa:bbb:ccc"
$ echo $aa
aaa:bbb:ccc:ddd:aaa:bbb:ccc
$ echo ${aa#a*:}
bbb:ccc:ddd:aaa:bbb:ccc
$ echo ${aa##a*:}
ccc
```
${變數%關鍵字} : 若變數內容從尾向前的資料符合『關鍵字』，則將符合的最短資料刪除  
${變數%%關鍵字} : 若變數內容從尾向前的資料符合『關鍵字』，則將符合的最長資料刪除  
```
aa="aaa:bbb:ccc:ddd:aaa:bbb:ccc"
$ echo ${aa%:b*}
aaa:bbb:ccc:ddd:aaa
$ echo ${aa%%:b*}
aaa
```

---
### :arrow_heading_down:Bash 的運算和特殊符號  
![](https://i.imgur.com/cWkiv8u.png)  

![](https://i.imgur.com/yN8gz0e.png)  

![](https://i.imgur.com/ASLZzhM.png)  

![](https://i.imgur.com/JJf6yVc.png)  





 

