# Bash script 的語法命令
## :wave: 課前須知  
:book:  
**read**  
參數:  
-p：指定讀取值時的提示字元(沒打變數名稱時，系統預設為 \$REPLY)  
-t：指定讀取值時等待的時間(秒)   
-n1 : 只接收一個字元就退出(不用案enter)  
-s : 使輸入的資料不顯示出來(實際上資料顯示出來的顏色與背景同色)  
常用在需要問答(選單)的時候  

:book:  
**/dev/null: /dev/null 在 Unix 或 Linux 就像黑洞, 會將任何導入的東西吃掉, 簡單來說就是程式會照常執行, 但不會輸出任何執行結果。**  
**1-標準輸出 ， 2-錯誤輸出 ， &=1+2**
```
bigred@gw:~$ cat wprog11.sh  1>out1 2>out2
bigred@gw:~$ cat wprog11.sh  &>/dev/null  
```
:book:  
declare -i 變數 : 宣告變數為整數  
(之後如果要做運算時會比較方便)  

:book:  
**區域變數 : 程式內變數各自獨立不往上，也不往下傳**
**export** :   
用於把變數變成當前shell和其子shell的環境變數，存活期是當前的shell及其子shell  
因此重新登陸或者關閉當前shell及其子shell後，它所設定的變數就消失了。  
**source** :  
作用:在當前bash環境下讀取並執行FileName中的命令，使環境變數生效。  
該命令可用命令“.”來替代。  
```
bigred@ds159:~$ cat test  
#!/bin/bash  
echo testbegin  
echo $a1  
export a1="0"  
echo test.a1 ${a1}  
a2="00"  
echo test.a2 ${a2}  
=================================  
~$ ./test  
testbegin  

test.a1 0  
test.a2 00  
test1 begin  
0  
test1.a1 1  
test1.a2 11  
=================================  
~$ export a1="up"  
~$ ./test  
testbegin  
up  
test.a1 0         
test.a2 00    
test1 begin  
0    
test1.a1 1  
test1.a2 11  
    
~$ echo $a1  
up   
```

:book:  
**自訂函數**
函數名稱{ 命令;  }  
函數名稱(){命令；}  
function 函數名稱 { 命令;  }  
function 函數名稱() { 命令;  }  
```
bigred@bean:~$ cat func04
#!/bin/bash
sayfun()
{
echo testfun$1
arg=$1
}

echo 111
echo $arg
sayfun
sayfun  abc
echo 222
echo $arg

bigred@bean:~$ ./func04
111

testfun
testfunabc
222
abc
```
 






---

## :gift: IF 使用方法
:o: **格式一**   
If   [  條件判斷式 ]  ;  then  
	當條件判斷式成立時，可以進行的指令工作內容  
fi     <==將 if 反過來寫，結束 if 之意！  
```
#!/bin/bash
clear
read -p "do you love me ?" ans
if [ "${ans,,}" = "y" ] ; then
 echo "i love you too"
fi
if [ "${ans,,}" = "n" ] ; then
  echo "i hate you"
fi
```
:o: **格式二**   
if 條件; then  
    語句  
else  
    語句  
fi  
```
#!/bin/bash
clear
read -p "do you love me ?" ans
if [ "${ans,,}" = "y" ]
then
 echo "i love you too"
else
  echo "i hate you"
fi
```
:o: **格式三**
if [ 條件判斷一 ] && ( || ) [ 條件判斷二 ]; then       
    執行內容程式   
elif [ 條件判斷三 ] && ( || ) [ 條件判斷四 ]; then     
    執行第二段內容程式   
else                                            
    執行第三段內容程式   
fi  
```
#! /bin/bash  
read  -p  "no?"  ans  
if  [ ${ans}  -le  60 ] ;then  
echo  "D grade "  
elif   [ ${ans}  -le  80 ] ; then  
   echo  " C grade "  
elif   [ ${ans}   -le  90 ] ; then  
    echo  " B grade "  
else  
echo " A  grade "  
fi  
```


---
## :cake: Case 使用方法  
:o: **格式**  
case  $變數名稱 in  
  "第一個變數內容")  
	程式段  
	;;   
"第二個變數內容")  
	程式段  
	;;  
     *)   
	不包含第一個變數內容與第二個變數內容的其他程式執行段  
	exit 1  
	;;  
esac    
```
#!/bin/bash  
clear  
read -p "你喜歡川普嗎(y/n)?"  number  
case $number in  
 y | Y)  
        echo "川普粉絲多一人"  
        ;;  
 n | N)  
        echo "討厭川普又多一人"  
        ;;  
*)  
        echo "請輸入(y或n)?“  
;;  
esac  
echo "謝謝您的答案"  
```


---
### :four:  FOR 迴圈  
:o: **格式一**
for 變數 in 參數  
do  
     statements  
done  
```
bigred@bean:~$ ./in08
bigred@bean:~$ ./in08 one two three four
one
two
three
four
bigred@bean:~$ cat in08
#!/bin/bash
for var in  $@
do
echo  $var
done
```

:o: **格式二**  
for (( 初始值; 限制值; 累進值 ))
do    
     statements    
done   
```
bigred@bean:~$ ./for07 10 20
begin
sum 10 到 20  總和為165
===
bigred@bean:~$ cat for07
#!/bin/bash
echo "begin"
 [-lt 2 $#  ] && echo "要給2個值" && exit
star=$1
end=$2
sum=0
for ((no=${star};no<=${end};no=no+1))
do
   sum=$((sum+no))
done
echo "sum $star 到 $end  總和為$sum"
```


---
### :dizzy: While 迴圈 
![](https://i.imgur.com/v0isAhl.png)  

:o: **格式一**  
while [ <some test> ]  
do  
<commands>  
done  
```
#!/bin/bash
sum=0
no=1
while [ "${no}" -lt 4 ]
do
echo ${no}
sum=$((sum+no))
no=$((no+1))
done
echo  $sum
```
:exclamation: [ <some test> ] 也可以寫成 ture  (一直跑就對了)
```
#!/bin/bash

while true
do
clear
echo  -e "
 1- file name
 2- exit"
#echo   "1- filename"
#echo   "2- exit"
read -p "choice?" cho
case $cho in
1)
 read -p "file name?" fn
 [ -f $fn ] && break
 echo "file not found" && sleep 1 && continue
;;
2)
 break
;;
*)
 read -p  "please choice 1 or 2 !" -t 2
 sleep 1
;;
esac

done
echo "program end"
```
:o: **格式二**  (對檔案做逐行讀取)     
:a:   
cat 檔案 | while read 變數
do
<commands>  
done
```
#!/bin/bash
count=1    # 設定陳述式，不加空格
cat txt01 | while read var      
# cat 命令的輸出作為read命令的輸入,read讀到>的值放在line中
do
   echo "Line $count:$line"
   count=$[ $count + 1 ]          # 注意中括弧中的空格。
done
echo "finish"
exit 0
```
:b:     
while read 變數  
do  
<commands>    
done < 檔案    

```
#!/bin/bash
while read   var
do
   echo "$var" ;
done <  txt01
```


---
### :rabbit: Select in 迴圈
:o: **格式二** 
select variable [in list] ;  
do  
    循環體命令  
done  
```
#!/bin/bash
clear
echo "請猜一下我喜歡的水果?
 輸入以下(1-4)"
select var in 蘋果 梨子 橘子 西瓜
do
  echo "您输入的内容为：$REPLY"
  if [ $REPLY = "1" ] ; then
    echo "恭喜! 你答對了!!"
    break
  else
    echo "答錯了!再猜一次"
  fi
done
echo 本程式結束
```


---
### :ghost: 陣列  
設定陣列內容  
方法一:  
array名稱[0]=var0內容  
arrya名稱[1]=var1內容  
arrya名稱[2]=var2內容  
    …  
array名稱[n]=varN內容  
  
方法二:  
array名稱=(var0內容  var1內容  var2內容  var3內容 … varN內容)  
  
方法三:  
array名稱=([0]=var0內容  [1]=var1內容  [2]=var2內容 … [n]=varN內容)  
```
#!/bin/bash   
fruits=(   
  “蘋果"  
  “梨子"  
  “橘子"  
  “西瓜"  
)   
echo “請猜一水果"    
select var in ${fruits[@]}   
Do  
  echo "您输入的内容为：$REPLY"   
  if [ $var = “蘋果" ] ; then  
    echo “恭喜 答對了!"  
    break  
  else  
    echo “再猜一下"  
  fi  
Done  
Echo 結束  
```


---
### :spiral_note_pad: 綜合範例  
```
bigred@bean:~$ cat func07  
cpu()  
{  
cpuinfo=$(cat /proc/cpuinfo|grep "model name"|head -n 1|cut -d ":" -f 2)  
}   
memory()  
{  
memoryinfo=$(free -mh |grep "Mem:"|fmt -u | cut -d " " -f 2)  
}  
hd()  
{  
hdinfo=$( df -h |grep "dev/sda2"|fmt -u|cut -d " " -f 2)  
}  

bigred@bean:~$ cat menu2  
#!/bin/bash  
export  cpuinfo  
export  memoryinfo  
export  hdinfo  
source   func07    
while true  
do  
clear  
echo "  1.cpu  
  2.memory  
  3.HD  
  4.exit"  
read -p "choice?" cho  
case  $cho in  
1)  
cpu  
./cpu1  
;;  
2)  
memory   
./memory1  
;;  
3)  
hd  
./hd1  
;;  
*)  
break  
;;  
esac  
done  
```