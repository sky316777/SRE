test

![](https://i.imgur.com/Xy5lNjI.png)
![](https://i.imgur.com/UvCdXvl.png)
![](https://i.imgur.com/gig1J9p.png)
![](https://i.imgur.com/2hMWg4z.png)
![](https://i.imgur.com/rkpuHGy.png)
![](https://i.imgur.com/qOUgnDK.png)
![](https://i.imgur.com/PljBiHm.png)
![](https://i.imgur.com/AkHyfbV.png)
![](https://i.imgur.com/3PIIpm5.png)
![](https://i.imgur.com/UGzGRRM.png)
![](https://i.imgur.com/53lFoq0.png)
![](https://i.imgur.com/wztzZxH.png)
![](https://i.imgur.com/a0GdRTA.png)
![](https://i.imgur.com/prpibXt.png)
![](https://i.imgur.com/scmRsMn.png)
![](https://i.imgur.com/EtIiKyW.png)
![](https://i.imgur.com/fJAcNIH.png)
![](https://i.imgur.com/akaXFTg.png)
![](https://i.imgur.com/ghmwjU0.png)
![](https://i.imgur.com/RSAC43a.png)
![](https://i.imgur.com/dpRg7dk.png)
```
--COGNOS QUERY
--STRUCTURE,1,1
--DATABASE,TPDB03-EUC
--DATASOURCENAME,C:\Documents and Settings\kim.hu\桌面\20191101-20201031.imr
--TITLE,20191101-20201031
--BEGIN SQL
SELECT  
       T1."CHDRNUM"  as 保單號碼,
	   T1."COVERAGE" as 險種序號,
	   T1."CRTABLE" as 險種簡稱,
	   T1."PLENG" as 險種英文簡稱,
	   T1."PTYPE1" as Stat_Fund,
	   T1."PTYPE2" as Section,
	   T1."PTYPE3" as Sub_Section,
	   T1."NSUMINS" as 保額,
	   T2."OLDPREM" as 原保費,
	   T2."NOWPREM" as 現在總保費,
	   T1."NINSTAMT" as 基本保費,
	   T1."NBILLPREM" as 應收保費,
	   T1."NANNPREM" as 年繳化保費,
	   CAST(SUBSTRING(CAST(T1.CRRCD as varchar) ,1,4) +'-' + SUBSTRING(CAST(T1.CRRCD as varchar) ,5,2) +'-' + SUBSTRING(CAST(T1.CRRCD as varchar) ,7,2) AS DATE) AS 起保日期 ,
	   T2."NBILFREQ" as 繳別,
	   T1."PCESTRM" as 繳費年期,
	   T1."STATCODE" as 險種狀況,
	   T1."PSTATCODE" as 繳費狀況,
	   T2."CNTCURR" as 幣別,
	   T2."SRCEBUS" as 業務來源,
	   T2."UNITNAME" as 單位名稱,
	   T1."LIFESEX" as 被保人性別,
	   T1."LIFEAGE" as 被保人年齡,
	   T2.OWNER_ID  as 要保人ID,
	   T2."MAGE" as 要保人年齡
from  SSADB.[dbo].[BDS_D_UC01_H]  T2 left outer join SSADB.[dbo].[BDS_D_UC02_H] T1 on (T2."CHDRNUM" = T1."CHDRNUM") and (T2.[SYS_FROM] = T1.[SYS_FROM] AND T2.[DATA_DATE] = T1.DATA_DATE)
WHERE 
CAST(SUBSTRING(CAST(T1.CRRCD as varchar) ,1,4) +'-' + SUBSTRING(CAST(T1.CRRCD as varchar) ,5,2) +'-' + SUBSTRING(CAST(T1.CRRCD as varchar) ,7,2) AS DATE) BETWEEN '2021-01-01' and  '2021-07-31'
AND T1.[DATA_DATE] = '2021-07-01'

--where T1.CRRCD >=  '2019-11-01' and T1.CRRCD <= '2020-10-31'

--END SQL
--COLUMN,0,substring (  保單號碼 , 1 , 6
--COLUMN,1,險種序號
--COLUMN,2,險種簡稱
--COLUMN,3,險種英文簡稱
--COLUMN,4,Stat. Fund
--COLUMN,5,Section
--COLUMN,6,Sub-Section
--COLUMN,7,保額
--COLUMN,8,原保費
--COLUMN,9,現在總保費
--COLUMN,10,基本保費
--COLUMN,11,應收保費
--COLUMN,12,年繳化保費
--COLUMN,13,起保日期
--COLUMN,14,繳別
--COLUMN,15,繳費年期
--COLUMN,16,險種狀況
--COLUMN,17,繳費狀況
--COLUMN,18,幣別
--COLUMN,19,業務來源
--COLUMN,20,被保人性別
--COLUMN,21,被保人年齡
--COLUMN,22,substring (  要保人ID , 1 , 4
--COLUMN,23,要保人年齡

```
