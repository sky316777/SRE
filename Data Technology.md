# Data Technology

![](https://i.imgur.com/Gr17SeF.png)

* Hadoop 由多台電腦構成，也是一個叢集(有容錯/穩定度高)
資料來源有各式各樣的資料型態多樣 ex. 國家的 opendata(文字檔)/物聯網資料/企業的應用系統or資料庫系統…

:hand: 常見與 Hadoop 搭配的分析工具:  
1. HIVE  
使用 Hadoop 當作資料來源，用 SQL 命令做資料分析，分析的資訊可以提供給企業當作決策報告  
2. Spark  
Spark (java開發的)的資料來源也是從 Hadoop 來，是叢集架構也是用在 AI ，目前也有企業用 spark 在做資料分析，分析的資訊可以提供給企業當作決策報告  
\>> spark 用 scala language 當作分析語言(Spar 也可加入 python/golan)  
3. Hbase  
Hbase 也是叢集系統，是 NoSQL 資料庫(database&table)
資料都存在 Hadoop 裡面。使用的領域比較多在 IOT 的領域，可作為 IOT 的資料庫(應付資訊的暴衝進與出)
\>> Hadoop 若消失，Hbase 就也不存在了

---

![](https://i.imgur.com/7I5Hmug.png)
### Raw data 原始資料 : senser 送進來的資料不能做改變

* Velocity 資料輸入輸出速度  
資料的傳輸流動是非常快速的，許多資料要能即時得到結果才能發揮最大的價值，因此也有人會將 Velocity 認為是時效性  
* Variety 資料類型  
大數據的來源種類包羅萬象，要小心資料是否真的是正確  
* Volume 資料量  
大數據平台的系統資訊是慢慢成長的，所以是在 Hadoop 一開始部屬的時候先行預設空間的容量ex. 使用單位(T)  
* Veracity 真實性  
資料要經過整理、確認才可以真正的使用  

---

### Hadoop 架構中最重要的4種模組:  
1. Command  
2. HDFS  
3. YARN  
4. Mapreduce  

![](https://i.imgur.com/SpyTtB7.png)  

* MapReduce 資料處理引擎  
* YARN 分散式運算系統  
Resource Manager 與 Node manager (淺藍黃)組成，用來跑資料分析程式 mapreduce 所寫的program(Hive/Spark)。  
* HDFS 資料儲存檔案系統  
Active,Secondary Name Node 與 Data Node (綠紅藍)組成 hadoop 的 HDFS 檔案系統。  
檔案的 metadata 會透過 Active Node 存到 master 機器上。  
檔案的內容則會透過 Date Node 儲存在 worker 機器上。   

以上的 Node 和 Manager 都是 java 程式，紅綠那排的都會在 master 主機上面跑，下面的藍黃組合則會在 worker 機上面跑。  

---

##  HDFS 底層運作架構  
#### 存入 HDFS 的資料是不能更改也不能做局部修改的，所以必須是重新全部刪除再新增(主要是因為raw data的資料不可以更動)

### :writing_hand: HDFS 分散檔案系統 - 寫入檔案  
![](https://i.imgur.com/NdbXEMd.png)

* 在 ADM 主機 Hadoop client，打 wget 下載需分析的資料檔後，將這個檔案放到 HDFS 前，在 client 會把資料先進行切割，預設為每 128MB 單位做切割。  
* 資料在做儲存之前，要先讓 namenode 知道有幾個機櫃資源可以使用，他就會做資料分配到不同機櫃。因為機櫃也有可能會壞，這樣的作法是要提高容錯能力 

![](https://i.imgur.com/0NKcTlu.png)

* file.txt 被切割成 3 個 block，client 會先問 namenode 資料區塊要存在哪裡。namenode 會提供儲存清單，說明不同 block 存入在哪些 datanode 中。client 接受到回復之後會把 blockA 放到 datanode1 中，再複製送到 datanode5 再送到 datanode6。
\>> **代表這三台機器會存有相同的 blockA 資料(這個是 replicate 可以讓很多人可以同時讀取檔案，不叫做 backup)**  

### :book: HDFS 分散檔案系統 - 讀取檔案  
![](https://i.imgur.com/SMDchPh.png)

* client 會問 namenode results.txt存在哪，namenode 會提供儲存清單，說明不同 block 分別在哪些 datanode 中。
client 會根據 namenode 回覆的資料依照順序去讀取資料(直的順序):  
A> datanode1,5,6  
B> datanode8,1,2  
C> datanode5,8,9  
:star_of_david: **HDFS 可以允許多人同時讀取檔案，並做分析這叫做 Round-robin (循環式服務)**   
例如:  
Tim: 1/8/5 ; 5/1/8 ; 6/2/9  
David: 5/1/8…  
Mandy: 6/2/9…  

## HDFS 常用操作命令範例
```
$ hdfs dfs -ls /sre
$ hdfs dfs -mkdir -p /sre/delta/sre200/dip
$ hdfs dfs -put opendata/dip/all/*.csv  /sre/delta/sre200/dip
$ hdfs dfs -put -f opendata/dip/all/*.csv  /sre/delta/sre200/dip
$ hdfs dfs -cp -f  /sre/echo/sre301/radio.txt /sre/delta/sre200/
$ hdfs dfs -appendToFile data.txt /sre/delta/sre200/data.txt
$ hdfs dfs -cat /sre/delta/sre200/data.txt

$ hdfs fsck  /sre /delta/sre200/passwd  -files  -blocks  -locations
Connecting to namenode via http://m31:50070/fsck?ugi=sre100&files=1&blocks=1&locations=1&path=%2Fsre%2Falpha%2Fsre100%2Fpasswd
FSCK started by sre100 (auth:SIMPLE) from /120.96.143.2 for path /sre/alpha/sre100/passwd at Wed Jun 09 01:31:21 CST 2021
/sre/alpha/sre100/passwd 5366 bytes, 1 block(s):  OK
0. BP-1466361734-120.96.143.31-1622864518101:blk_1073754588_13764 len=5366 Live_repl=3 
[DatanodeInfoWithStorage[120.96.143.56:50010,DS-3ee4048e-5589-425d-b9f3-84c80ce84303,DISK], 
DatanodeInfoWithStorage[120.96.143.52:50010,DS-7e253df5-e216-4ce7-bea2-021ea32b7452,DISK], 
DatanodeInfoWithStorage[120.96.143.51:50010,DS-16f0a5c0-1703-40af-891c-ada823b5cdbc,DISK]]
...........

#這個區塊(blk_1073754588_13764)在哪些 datanode(120.96.143.56/52/51)裡面;代表這三台機器存有相同的 block

```
---

## MapReduce 檔案型資料庫  

### MapReduce 處理流程  
![](https://i.imgur.com/GlTe874.png)  

* Map : 擷取/過濾  
Shuffle : 分類/排序  
Reduce : 分析/上載  

當執行 mapreduce 時就會跑上述流程，會將 HDFS 中的資料擷取、分類、分析，最後再上載回去 HDFS 中  

### Mapreduce 叢集運作架構圖  
![](https://i.imgur.com/g9rOQCy.png)  

* Split : 可以為一個或多個 block
淡藍色框框為 worker
白框框為 disk

當我要對檔案進行資料分析時，map 去找存放此檔案的 worker 上執行(此圖的檔案剛好被分成 3 個 split 放在不同的 worker)，白色區塊為擷取/過濾後的資料且存在硬碟，透過 shuffle 使用網路傳送至另一台 worker 機，此 worker 機的 shuffle 會再將所有白色區塊進行合併分類排序，最後透過 reduce 分析並上載到 HDFS 檔案系統中。  

:arrow_right: 三個 map 程式碼都一樣， Mapreduce 把我們寫的一個程式在三台電腦裡面同時 run 且處理不同的資料  

---

## YARN 分散運算系統  

### YARN 分散運算系統運作圖  
![](https://i.imgur.com/oQCMr7B.png)  

1. 使用者在 client 機打 SQL 指令，透過 Hive 轉成 mapreduce java 程式
2. 透過網路丟給 resource manager，請他執行程式
3. resource manager 指派 Mapreduce Application master (worker機)去執行程式
4. MRAM先問 Namenode 要處理的 block 的位置清單(datanode)
5. 接收到訊息後 MRAM 會找 RM 說要在哪些 worker 機跑 map 程式
6. 透過 Nodemanager 做 map (過濾擷取)產生的小白區塊，透過 shuffle 用網路傳送彙整到有 reduce 的機器上面
7. 將小白區塊的資料 merge (分類排序)成大白區塊在做 reduce 分析後做上載到 HDFS

---

![](https://i.imgur.com/ICwas45.png)  
* 每一台 worker 機會回報給 Rm 有多少 cpu/ram 可以用  
* map 可以用的 ram 大小是根據實際資料量自訂出來的(擷取/過濾 不需要太大的 ram 空間)  
* reduce 是分析，需要比較複雜的運算處理，所以需要多一點的 ram 空間  
* Yarn 有一個 FIFO 的規則，若今天第一個使用者就需要 24G/24cores，則 Yarn 就會直接給第一個人它需要的所有資源，當第二個人進來時必須要等第一個使用者使用完釋放出資源後他才可以用。這部分可以改為 Fair 的設定，第一個使用者用完的閒置資源，會直接給第二個使用者使用，這樣就可以讓所有空閒資源被完善的使用。  

### 檢視 YARN 總量運算資源  
```
$ curl -s http://m32:8088/ws/v1/cluster/metrics | jq
{
  "clusterMetrics": {
    ......
    "availableMB": 57344,
    "allocatedMB": 0,
    "reservedVirtualCores": 0,
    "availableVirtualCores": 56,
    "allocatedVirtualCores": 0,
    "containersAllocated": 0,
    "containersReserved": 0,
    "containersPending": 0,
    "totalMB": 57344,
    "totalVirtualCores": 56,
    "totalNodes": 7,
    .......
  }
}
[重點] YARN 系統內定每部 Node Manager 的記憶體為 8G，CPU 為 8 Core
```
### 設定單一 Node Manager 運算資源  
:arrow_right: 存在 worker 機上面  
```
$ cat /opt/hadoop-2.10.1/etc/hadoop/yarn-site.xml
......
 <property>
    <name>yarn.nodemanager.resource.memory-mb</name>
    <value>8192</value>
  </property>
  <property>
    <name>yarn.nodemanager.resource.cpu-vcores</name>
    <value>8</value>
  </property>
.......
```
### 設定單一 Container 可用運算資源  
:arrow_right: resource manager 在讀的，存在 m32(老師的) 機器  
**yarn-site.xml 同時也是設定 FIFO 變成 fair 的地方**  
```
$  cat /opt/hadoop-2.10.1/etc/hadoop/yarn-site.xml
........
  <property>
    <name>yarn.scheduler.minimum-allocation-mb</name>
    <value>384</value>
  </property>
  <property>
    <name>yarn.scheduler.maximum-allocation-mb</name>
    <value>896</value>
  </property>
  <property>
    <name>yarn.scheduler.minimum-allocation-vcores</name>
    <value>1</value>
  </property>
  <property>
    <name>yarn.scheduler.maximum-allocation-vcores</name>
    <value>1</value>
  </property>
..........
```
### 設定 MapReduce Job 的運算資源  
:arrow_right: adm (client) 會去讀的  
```
......
   <property>
      <name>yarn.app.mapreduce.am.resource.mb</name>
      <value>512</value>
   </property>
   <property>
      <name>yarn.app.mapreduce.am.command-opts</name>
      <value>-Xmx384m</value>
   </property>
   <property>
      <name>mapreduce.reduce.memory.mb</name>
      <value>896</value>
   </property>
   <property>
      <name>mapreduce.reduce.java.opts</name>
      <value>-Xmx512m</value>
   </property>
   <property>
      <name>mapreduce.map.memory.mb</name>
      <value>512</value>
   </property>
   <property>
      <name>mapreduce.map.java.opts</name>
      <value>-Xmx384m</value>
   </property>
```
### 資源設定衝突的時候
```
設定 reduce 記憶體需求
$ sudo nano /opt/hadoop-2.10.1/etc/hadoop/mapred-site.xml
........
   <property>
      <name>mapreduce.reduce.memory.mb</name>
      <value>1024</value>
   </property>
........

$ yarn  jar  /opt/hadoop-2.10.1/share/hadoop/mapreduce/hadoop-mapreduce-examples-2.10.1.jar pi 2 100000
............
2021-02-05 13:32:57,099 INFO mapreduce.Job: Job job_1612530333151_0006 failed with state KILLED due to: REDUCE capability required is more than the supported max container capability in the cluster. Killing the Job. reduceResourceRequest: <memory:1024, vCores:1> maxContainerCapability:<memory:896, vCores:1>

[註] 因 YARN 的 Container 最大記憶體為 896Ｍ，無法滿足 reduce 的要求，使用 hadoop jar 命令執行上述命令，如執行失敗，不會出現錯誤訊息
```

* yarn-site.xml 是 RM(M32)在讀取的檔案，這是主要分配資源的一個角色，所以必須以他的設定檔內容當作資源執行的分配檔。
* mapred-site.xml 是 Hadoop client 端主機(ADM)在讀取的檔案，所以在這個檔案的資源內容不可以超越 yarn-site.xml 的資源，不然 RM 沒那麼多資源可以給 client 使用。

---

## bzip2  

* 介紹: 儲存企業重要資訊: Hadoop 備份系統直接是用壓縮檔做分析，但前提是實體機的 CPU 一定要選比較好的才可以   
原本一個 100G 檔案，若用 bzip2 存他會變成 block(一塊，9k)的形式，所以一個區塊一個區塊的解壓縮(跟 7zipp 必須把檔案一次讀完才可以解壓縮是不同的)  
Bzip2 本身裡面就是一塊一塊的壓縮檔, 存到 HDFS(每128M存成一個block)可以讓 DN 解壓縮&並做擷取過濾分析( Map 程式可做單獨某一區塊的資料做處理)  

```
$ hadoop fs -mkdir -p mydataset/sales

$ wget http://eforexcel.com/wp/wp-content/uploads/2017/07/1000000%20Sales%20Records.zip

$ unzip '1000000 Sales Records.zip'

$ bzip2  -1 -kz  '1000000 Sales Records.csv' -c  > sales.bz2
 -z --compress    force compression
 -k --keep        keep (don't delete) input files
 -1 .. -9         set block size to 100k .. 900k

$ dir sales.bz2 
-rw-r--r-- 1 bigred bigred 30M  7月 28 09:14 sales.bz2

$ hadoop fs -D dfs.block.size=16m -put  sales.bz2  mydataset/sales/

$ hdfs  fsck  mydataset/sales/sales.bz2
......
 Total blocks (validated):	2 (avg. block size 15386158 B)
........
```
參數解說:
1. wget 下來的檔案如果是壓縮檔要先解壓縮才可以用 bzip2
2. bzip2  -1 -kz
-1: 100k 為一個 block
-k: 保留壓縮檔
-z: 壓縮
3. 針對特定檔更改儲存大小(128m 改成用16m 存)會有 2 個 block (30m/16m)

---

## Hive 資料倉儲工具

:star2: 傳統關聯式 SQL 與 分散式 SQL 的差別  
**(IT) 傳統關聯式 SQL : schema on write**  
要先建立 table (資料型態)再把資料相對應的值 insert into 進去  

**(Big Data) 分散式 SQL (ex.Hive) 作法跟傳統做法完全顛倒 :  schema on read**  
先把資料寫在一個檔案，再 create table 套進去資料裡面
就可以分析資料了  

### :hibiscus: Hive 指令   

$ hive -S
(也可以不加 -S 進入 hive)

$ hive -S -f customers.hsql
(在 hive 裡面創建表格，類似 kubectl apply -f )

$ hive -S -e "show tables"
( hive -S -e "要在 hive 裡執行的指令")

```
進入 Hive 後，在裡面可以輸入 SQL 和 dfs 指令，以下簡易範例
hive (default)> create table dummy (value string);
hive (default)> insert into dummy values('abc');
hive (default)> insert into dummy values('xyz'); 


hive (default)> dfs -ls /user/<Account>/hive/dummy;
Found 2 items
-rw-r--r--  2 rbean rbean  4 2020-09-27 01:40 /user/<Account>/hive/dummy/000000_0
-rw-r--r--  2 rbean rbean  4 2020-09-27 01:41 /user/<Account>/hive/dummy/000000_0_copy_1
#每insert into一筆就會產生一個檔案


hive (default)> drop table dummy;
hive (default)> dfs -ls /user/<Account>/hive/dummy; 
ls: `/user/<Account>/hive/dummy': No such file or directory
#內部資料表被刪掉, 這個目錄也會直接被刪除掉
```

:arrow_right: **第一次建立表格，Hive 會在 HDFS 建立 user/<account>/hive 目錄**，用來儲存資料表的資料。  
因為**每新增一筆資料就會在 HDFS 中產生一個檔案**，但這是一個 rawdata 所以得知 Hive 是無法執行刪除資料這個指令 & 修改內容。**若要刪除某筆資料是不行的，只能把整個 table 刪除掉**

```
echo $'1,Ramesh,32,Ahmedabad,2000.00
2,Khilan,25,Delhi,1500.00  
3,kaushik,23,Kota,2000.00 
4,Chaitali,25,Mumbai,6500.00
5,Hardik,27,Bhopal,8500.00 
6,Komal,22,MP,4500.00
7,Muffy,24,Indore,10000.00' > customers.csv

$ hdfs dfs -mkdir -p mydataset/customers

$ hdfs dfs -put customers.csv  mydataset/customers

echo $'CREATE TABLE customers (
   id int,
   name string,
   age int,
   address string,
   salary float
)
ROW FORMAT DELIMITED FIELDS TERMINATED BY \',\'
STORED AS TEXTFILE LOCATION \'mydataset/customers\' ; ' > customers.hsql
(建立表格型態的檔案)
```

參數解說:  
1. -put: 把檔案傳到 HDFS 中  
2. CREATE TABLE customers : 建立**內部資料表**  
3. Table name(customers) 不一定要跟下面的路徑名(mydataset/customers)一樣  
4. 'mydataset/customers\' 這個路徑是 customer.csv 存的目錄 mydataset/customers，這個是 Hive 建立，也可以自己手動建喔

### Hive 外部資料表  

**外部資料表建立時必須非常清楚指名他在哪個資料夾，這個資料夾必須事前建立，而不能由 Hive 建立**
若要刪除這個資料表，只有當前使用者的資訊會被刪除，並不會影響到其他使用者
```
$ myp.sh 
p102.csv ok
p103.csv ok
p104.csv ok
p105.csv ok
p106.csv ok
p107.csv ok
p108.csv ok

$ hdfs dfs -mkdir  -p  mydataset/twpop/

$ hdfs dfs -put opendata/p/*.csv   mydataset/twpop/

$ echo $'CREATE EXTERNAL TABLE twpop (
statistic_yyy int, 
site_id string,
people_total float,
area float,
population_density int
)
ROW FORMAT DELIMITED FIELDS TERMINATED BY \',\'
STORED AS TEXTFILE LOCATION \'mydataset/twpop\';' > twpop.hsql

$ hive -f  twpop.hsql  2>/dev/null

$ hive -e 'select count(*) from twpop where statistic_yyy=104'  2>/dev/null
370
#count這樣的作法在傳統SQL是傷害資料庫的事情

```
參數解說:  
echo $'CREATE EXTERNAL TABLE twpop : 建立**外部資料表**  
**echo 的指令是在該位使用者家目錄執行，這個 schema 不會跟其他使用者共用，且資料存在HDFS中**  

### Hive Partition  
partiotion 這個指令可以針對特定的欄位做切割，找資訊的時候就不用再去看整個大表的其他內容，可以提升讀表的速度  

```
$ echo $'CREATE EXTERNAL TABLE twpop_yp (
site_id string,
people_total float,
area float,
population_density int
)
PARTITIONED BY (year string)
STORED AS TEXTFILE LOCATION \'mydataset/twpop_yp\';' > twpop_yp.hsql

$ hive -f twpop_yp.hsql  2>/dev/null

[註] 上面命令會自動建立 mydataset/twpop_yp 目錄

$ hive -e 'show tables' 2>/dev/null
twpop
twpop_yp
```

:arrow_right: twpop_yp.hsql 創造一個表格，有五個欄位，利用 year 欄位做分割  
用年份做分類就可以只單獨看 year=102 不用看其他 year=103 or 104…

```
$ echo 'SET hive.exec.dynamic.partition.mode = nonstrict;
INSERT OVERWRITE TABLE twpop_yp 
PARTITION (year)
SELECT site_id,people_total,area,population_density,statistic_yyy
FROM twpop; ' > pp_insert.hsql

$ hive -f pp_insert.hsql  2>/dev/null

$ hdfs dfs -ls mydataset/twpop_yp
Found 7 items
drwxr-xr-x  - bigred bigboss 0 2020-09-27 07:09 /dataset/twpop_yp/year=102
drwxr-xr-x  - bigred bigboss 0 2020-09-27 07:09 /dataset/twpop_yp/year=103
drwxr-xr-x  - bigred bigboss 0 2020-09-27 07:09 /dataset/twpop_yp/year=104
drwxr-xr-x  - bigred bigboss 0 2020-09-27 07:09 /dataset/twpop_yp/year=105
drwxr-xr-x  - bigred bigboss 0 2020-09-27 07:09 /dataset/twpop_yp/year=106
drwxr-xr-x  - bigred bigboss 0 2020-09-27 07:09 /dataset/twpop_yp/year=107
drwxr-xr-x  - bigred bigboss 0 2020-09-27 07:09 /dataset/twpop_yp/year=108

$ hive -e 'select * from twpop_yp where year=102 limit 2' 2>/dev/null
新北市板橋區　	556920.0	23.1373	24070	102
新北市三重區　	389813.0	16.317	23890	102
```
:arrow_right: twpop (有資料的大表) ; twpop_yp(空表)把舊表切割完放在下面 

1. mode = nonstrict 預設是靜態，但這邊改為動態分割
代表 year 產生這些的目錄不用特別做宣告 (ex. year=102/103/104…這種動作)  
2. hdfs dfs -ls mydataset/twpop_yp 這些目錄下面有一個檔案，裡面的資料就是 y102 的資料，裡面的欄位是沒有年份的，因為這邊就很明確是用年份去做切割  
3. 大表(twppop)欄位名稱跟小表(twpop_yp)欄位做比對，遇到不同的欄位以它為準就做切割(年分)，所以這邊才是用不同的年份顯示   
year=102 : 用宣告的名稱(year)當作目錄名稱的開頭  
statistic_yyy 會帶出年分  
4. 資料顯示是取決於 pp_insert.hsql 中 SELECT 的順序  

### Hive Parquet  

Partioion 是橫切讀一整列 (對欄位分類產生目錄/資料夾) ; Parquet 是直切去讀取資料 (存成檔案，在 partition 目錄之下)  
**傳統是全部資料都要掃過一遍，若是 partition+parquet 就可以直接命中找到鎖定的資訊**  

![](https://i.imgur.com/qUpeqkw.png)   
Ex. 要找 select C where A=A2 所有的資料做完 partition 後，則會馬上找到是 C2 ; parquet 可以直接找 A2，不會再去找A1/A3，對應到 C 就是 C2  

```
$ echo $'drop table if exists twpop_par;
CREATE EXTERNAL TABLE twpop_par (
site_id string,
people_total float,
area float,
population_density int
)
PARTITIONED BY (year string)
STORED AS Parquet LOCATION \'mydataset/twpop_par\'; '> twpop_par.hsql

$ hive -f twpop_par.hsql  2>/dev/null

[註] 上面命令會自動建立 mydataset/twpop_par 目錄 
```

```
$ echo 'SET hive.exec.dynamic.partition.mode = nonstrict;
INSERT OVERWRITE TABLE twpop_par 
PARTITION (year)
SELECT site_id,people_total,area,population_density,statistic_yyy
FROM twpop; ' > ppp_insert.hsql

$ hive -f ppp_insert.hsql

$ hdfs dfs -ls mydataset/twpop_par
Found 7 items
drwxr-xr-x   - bigred bigboss    0 2020-09-27 08:03 /dataset/twpop_par/year=102
drwxr-xr-x   - bigred bigboss    0 2020-09-27 08:03 /dataset/twpop_par/year=103
drwxr-xr-x   - bigred bigboss    0 2020-09-27 08:03 /dataset/twpop_par/year=104
drwxr-xr-x   - bigred bigboss    0 2020-09-27 08:03 /dataset/twpop_par/year=105
drwxr-xr-x   - bigred bigboss    0 2020-09-27 08:03 /dataset/twpop_par/year=106
drwxr-xr-x   - bigred bigboss    0 2020-09-27 08:03 /dataset/twpop_par/year=107
drwxr-xr-x   - bigred bigboss    0 2020-09-27 08:03 /dataset/twpop_par/year=108

$ hdfs dfs -cat mydataset/twpop_par/year=102/000000_0
PAR1????,6(高雄市鼓山區　南投縣中寮鄉　?新北市板橋區.....
...........
# 與 partition 不同的是資料夾裡面的檔案內容 
```

### 比較原始，分割及 Parquet 三種資料集
```
原始資料集, 有 year 這欄位, 這欄位資料是重複
$ hdfs dfs -du -s -h mydataset/twpop/
107.7 K  215.4 K  mydataset/twpop

分割後的資料集, 沒有 year 這欄位, year 欄位變成 資料夾名稱
$ hdfs dfs -du -s -h mydataset/twpop_yp/
100.1 K  200.1 K  mydataset/twpop_yp

Parquet 格式的資料集, 因有壓縮所以會更小
$ hdfs dfs -du -s -h mydataset/twpop_par/
92.2 K  184.4 K  mydataset/twpop_par
```
參數解說:
du : 看目錄或檔案有多大  
-h: 給人看的懂得  
-s: 因為是目錄，所以要計算底下所有檔案的大小總合  









