# Kubernetes

k8s就是帶有自動化的作業系統，要做到免手動維運(hands- operation)  
服務對象: 企業的 application (erp/人資/會計...等系統)  

前提介紹:  
- Replication of components  
第一次就部屬xx個pod  
- Auto-scaling  
偵測到連接數量的暴增就會幫我們自動增加pod  
若後續不需要那麼多運算資源他也會自動回收pod，被回收的pod 資料會整個被收掉，他會先被收到其他的資料庫系統中。  
回收後的數量不會低於一開始發送的pod數量  
- Load balancing  
高效率資源分配，最佳資源化資源使用，誰有空誰可以服務  
- Rolling updates  
k8s幫程式設計師上版 or 降版；如果功能做錯，就會自動維運application  
- Logging across components  
- Monitoring and health checking  
監控pod (log資訊)並檢測他是否健康，
若application突然爆量，k8s會提供容錯功能讓機器在旁邊休息，休息完會恢復連線，不會當機  
- Service discovery  
DNS  
server解析電腦名稱，會動態修改電腦名稱或ip位置  
- Authentication  
用公私鑰作業 (bash script)  


---
## :page_facing_up: 目錄 :  

[1.指令集](https://)  
[2.pod](https://)  
[3.volume](https://)  
[4.deployment](https://)  
[5.service](https://)  
[6.帳號管理憑證](https://)  
[7.RBAC](https://)  
[8.context 遠端操作](https://)  

---

![](https://i.imgur.com/UaMOSwJ.png)  

架構解說:   
1. 人下達命令(kubectl)由API告知sch
2. 由sch分派pod要在哪個主機(node)上執行
3. Con-man 會做auto-scalling
4. Etcd=資料庫系統會記錄k8s整個運作資訊，若這個掛了，整個k8s就會消失了>>> 要做容錯(備份)

備註:  
\$ sudo crictl ps -a  
裡面有4個pod，裡面都只有1台container  

---

![](https://i.imgur.com/ejF1WN5.png)  

CRI: K8s要做出來，會透過CRI的標準產生pod  

CNI: k8s針對網路也有自訂出標準  
ex. flannel follow CNI標準，讓所有電腦主機/pod裡面透過static route(多台實體電腦的網路)可以互通有無  

CSI: 所有pod有自己專屬的檔案系統  

---

## K8s 的發展史  
![](https://i.imgur.com/pxvsfIU.png)  

K3s 用 containerd (有daemon)
K8s 用 CRI-O (有daemon)

---

## Pod 無法跨實體主機  
![](https://i.imgur.com/VmcRIqV.png)  

1. CRI 此介面系統，專門產生 pod  
2. pod 中的 container可以有多台，看需求去評估要多少 container  
3. Pod 不可以跨到另外一台主機 (node)，只會在單一主機 (node)上面跑  

---

## Pod 無法使用 localhost 互相連接  
![](https://i.imgur.com/zpWMWJI.png)  

多個pod 之間不可以透過localhost互連，但可用IP位置戶相連接(同網段，都在master的叢集裡);  
同個pod中的多個container可以用localhost戶相連接(因為共用網卡，同IP)  

---

## Pod 使用 NFS 做資料永存  
![](https://i.imgur.com/mHlMcwL.png)  

K8s自訂儲存標準:  
1. Cluster(多台電腦一個pod共用一個目錄區  
1GB(NFS中的目錄用mount過去pod)是用遠端的一台電腦(/var…1gb)，該電腦裡面有裝NFS(同windows網路上的芳鄰)就是把某一台的資料夾分享給其他電腦做使用  
2. NFS 可以想像成是一台 Qnap  

---

## Pod 可指定如何佈署到 Nodes
![](https://i.imgur.com/EEyl5HR.png)  
指定pod在哪個node之中執行  
先在實體主機上面貼標籤，若沒指定他會自行分配  

---

## Pod 可輕易做到高可用性 (HA)  
![](https://i.imgur.com/UvgYcoO.png)  

pod A replica=2 可以用 Depolyment Object  
1. 用相同規格產生，做HA的意思  
2. 當今天這兩個pod超過負荷使用量，這時候Auto-scalling會馬上再幫pod產生另外2個分身(依照一開始replica下的數量，而倍數產生)  

---

## Kubernetes Master 容錯  
![](https://i.imgur.com/GozvbL8.png)  
*建議企業建置至少要有容錯系統，這樣建置上才比較安全  

Majority: 若要執行完整的k8s的功能，至少要存活的master數量  
Failure Tolerance: 若要執行完整的k8s的功能，最多可以壞掉的master數量  
範例:  
五台 master若壞了三台，就不能新增/修改k8s pod，整個k8s功能被鎖死，但已經產生的pod可以繼續運作，壞掉的三台資料就消失  

---

## :earth_africa: kubectl 指令集  
\$ kubectl get all  
\$ kubectl get all -o wide   
\$ kubectl get nodes --show-labels   
\$ kubectl get po -o=custom-columns=NAME:.metadata.name,CONTAINERS:.spec.containers[*].name  
(列出現有的pod & container)  
\$ kubectl get pods -n kube-system  
\$ kubectl create -f  
\$ kubectl apply -f  
\$ kubectl delete -f  
\$ kubectl describe  
\$ kubectl exec  
\$ kubectl logs  
\$ kubectl edit   

---

## :sauropod: Pod  
### pod的內部結構  
![](https://i.imgur.com/C8HJTFp.png)  
pause : 裡面run的命令，sleep infinity(睡夢羅漢)，不容易被攻陷，把網卡等等放在裡面，卻能確保pod永存  

### pod 共用 PID namespace 的 yaml 檔  
```
apiVersion: v1
kind: Pod
metadata:
  name: sharepid
spec:
  shareProcessNamespace: true
  hostname: xyz 
  containers:
  - name: derby
    image: quay.io/cloudwalker/alpine.derby
    imagePullPolicy: Never
  - name: shell
    image: quay.io/cloudwalker/alpine
    stdin: true
    tty: true  
```

參數解說:  
kind: 宣告類型，在k8s產生pod  
metadata: 存在etcd資料庫中  
shareProcessNamespace: container共用namespace (PID)  
stdin: true / tty: true > 代表-it，產生虛擬終端機給貝殼程式用  

:arrow_right: imagePullPolicy有三種型態:  
Never: 每次重啟這個pod不會從網路上下載  
Always: 會從網路上下載  
IfNotPresent: 要依照該台node是否有此光碟片，沒有就去下載，但缺點是若今天光碟有新版本，指令會判定這台node有舊光碟就不會下載新光碟   


### pod 只定 namespace 且 container 共用 pid 範例  
```
kind: Pod
metadata:
  name: sharepid
  namespace: share
spec:
  shareProcessNamespace: true
  containers:
  - name: derby
    image: quay.io/cloudwalker/alpine.derby
    imagePullPolicy: Always
  - name: shell
    image: quay.io/cloudwalker/busybox
    stdin: true
    tty: true
  restartPolicy: Never  
  nodeSelector:  
    kubernetes.io/hostname : 120-96-143-156-m1
```

參數解說:   
namespace: 創立出來的pod在指定的namespace上面run  
restartPolicy: Never > 代表絕對不要重新啟動被刪除的container  
nodeSelector: 指定pod在哪個node上面執行  

:warning: pod 內定的 restartpolicy 是 always，所以Kubelet 會自動再啟動被關閉的 Container。  
一般來說在yaml檔建立出來的pod會在worker機上面執行，但此檔案指定在master主機上面run，因此imagePullPolicy才會是always(master主機上並沒有光碟檔)  


```
$ kubectl create namespace share
$ kubectl create -f sharepid.yml
$ kubectl exec -it sharepid  -c shell -- sh
(-c: container  
-- :對pod這個container下達的命令)

/ # ps aux
PID   USER     TIME  COMMAND
    1 root      0:00 /pause
    6 root      0:00 bash -c /derby/app/startup
   11 root      0:11 java -jar -Dderby.system.home=/derby/db /derby/app/dt-0.0.1-SNAPSHOT.war
   ............
/ # kill -9 11
/ # ps aux
PID   USER     TIME  COMMAND
    1 root      0:00 /pause 
   23 root      0:00 sh
   58 root      0:00 sh
   64 root      0:00 ps aux
/ # exit
```

:arrow_right: 如果有指定創立在特定的namespace一定要先建立namespace才可以建立ymal檔內容。  
上述PID 11 可以刪除成功是因為有別下指令不要重啟container  

---

## Liveness and Readiness Probes  
### Liveness Probe 示範 yaml 檔  
```
apiVersion: v1
kind: Pod
metadata:
  name: liveness-exec
spec:
  containers:
  - name: liveness
    image: k8s.gcr.io/busybox
    args:
    - /bin/sh
    - -c
    - touch /tmp/healthy; sleep 20; rm -rf /tmp/healthy; sleep 300
    livenessProbe:
      exec:
        command:
        - cat
        - /tmp/healthy
      initialDelaySeconds: 5
      periodSeconds: 5
```
:arrow_right: 解釋 :   
當 container 做事情做到太累了可能會當機，但他沒有死掉，所以他不會浴火重生。為此我們可以使用 Liveness Probe 主動偵測他是否還能工作，如果發現他當機了就把他重啟。  
上述例子是當 pod 把 /tmp/healthy 刪掉後進入 sleep，Liveness Probe 偵測到檔案不見後就會主動把他重啟。  


### Readiness Probe 示範 yaml 檔  
```
apiVersion: v1
kind: Pod
metadata:
  name: tcp-readiness
spec:
  containers:
  - name: tcp-readiness
    image: k8s.gcr.io/busybox
    stdin: true
    tty: true
    args:
    - /bin/sh
    - -c
    - sleep 20; timeout 20s nc -l localhost -p 9999; sleep 60; nc -l localhost -p 9999
    ports:
    - containerPort: 9999
    readinessProbe:
      tcpSocket:
        port: 9999
      initialDelaySeconds: 5
      periodSeconds: 5
```
:arrow_right: 解釋 :  
當 container 做事情做到太累了可能會當機，但他沒有死掉，所以他不會浴火重生。為此我們可以使用 Readiness Probe 主動偵測他是否還能工作，如果發現他目前無法工作就把他先放在旁邊，顯示沒有 ready 的狀態，直到偵測到它可以工作時再把它調整到 ready 狀態。  
上述例子是指當 pod 進行到 sleep 時，此時 Readiness Probe 偵測不到 port 號 9999，就會保護這個 pod 讓它顯示 not ready 不讓他被使用，等到 sleep 才讓它的狀態變 ready。  



---

## Multi-Container Pod Design Patterns

![](https://i.imgur.com/V8kNcvj.png)  

* Sidecar  
現在很多公司應用系統都是用瀏覽器啟動，這種web application一定都會有log檔 (詳細記錄)  
sidecar container會幫app container整理裡面的log檔(資料整理/備份/異地備援等)，他們可以透過localhost傳遞資料(穩定/安全/快速)  
* Adapter  
application 可以是多個container，adapter會整合這些系統，變成一個簡單的介面讓客戶使用。ex.客戶連到此網站，這個adapter會彙整整個企業的內部資訊，變成一個下拉式選單，點選自己想要的資訊  
* Ambassador  
根據App container的要求，Ambassador container會去外網蒐集資訊，然後整理好提供給App container  
ex.要取得opendata，幫程式設計師簡化設計流程

### Sidecar 示範 yaml 檔
```
apiVersion: v1
kind: Pod
metadata:
  name: webserver
spec:
  volumes:
    - name: shared-logs
      emptyDir: {}
  containers:
    - name: nginx
      image: quay.io/cloudwalker/nginx
      volumeMounts:
        - name: shared-logs
          mountPath: /var/log/nginx
    - name: sidecar-container
      image: quay.io/cloudwalker/busybox
      command: ["sh","-c","while true; do cat /var/log/nginx/access.log /var/log/nginx/error.log; sleep 30; done"]
      volumeMounts:
        - name: shared-logs
          mountPath: /var/log/nginx
``` 

參數解說 :  
volumes : 產生一個儲存空間，型態叫 emptyDir  
透過 get pod -o wide 看這個 pod 在哪台 node 執行，就會在特定的 node 中產生一個資料夾(目錄)，因為是 mount 進去，所以實際是在 emptyDir 上面作業。  
mountPath : emptyDir volume (shared-logs) 掛載(mount)到這個 nginx container 的目錄(/var/log/nginx)  

        
```
$ kubectl apply -f sidecar.yml 
pod/webserver created

$ kubectl get pod webserver -o wide
NAME        READY   STATUS    RESTARTS   AGE   IP             NODE   NOMINATED NODE   READINESS GATES
webserver   2/2     Running   0          10m   10.244.8.194   w1   <none>           <none>

$ curl http://10.244.8.194
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
```

:arrow_right: apply也可以產生pod，裡面的運作結構可以動態修改;
create出來的pod在運作中無法做修改  

---

## :mosque: Volume  
![](https://i.imgur.com/rOUwCbo.png)  
PV: 永存資料  
會在k8s裡面(此資料夾位置可由使用者指定，k8s在就存在，反之)


### Persistent Volumes
* Persistent Volume(PV) 由 storage 管理者負責產生，在 k8s 中就是一種可用的 storage resource，同樣也是一種 volume plugin，但有自己獨立的 lifecycle，且包含的就是實際與 storage 連結的實作細節。  

* Persistent Volume Claim(PVC) 則是來自使用者的 storage request，就跟 pod 一樣都是要消耗特定的資源，PVC 消耗 PV 資源(pod 消耗 node 資源)，PVC 指定特定 size or access mode 的 PV(pod 可以指定 CPU, memory … etc)。  


### pv 與 pvc 關係圖  
![](https://i.imgur.com/4aGdXh6.png)  
k8s 會把所創造出的 pv 視為一個群組，會根據 pvc 的要求(型態，容量大小)，給 pvc 他要的 pv。  
### pv 的種類圖  
![](https://i.imgur.com/cbccHCn.png)  


Access Modes 說明:  
**ReadWriteOnce (RWO)**: 一個 node 可 read-write  
**ReadOnlyMany (ROX)**: 一個 node 可 write，多個 node 可 read-only  
**ReadWriteMany (RWX)**: 多個 node 可 read-write  
三種存取模式提供掛載，但存取控制機制本身並非由 k8s 支援，而是由 PV resource provider 支援


### :arrow_right:  如果要在 node 保留 container 裡創造的使用者資訊，該如何使用 pv, pv-local ,pvc 方式去達成 ?  

以下為操作示範解答 :  
step 1 :
```
$ kubectl run t1 -it --image=quay.io/cloudwalker/alpine
$ kubectl exec -it t1 -- sh
/# apk update;apk upgrade -y
/# apk add openssh
/# scp -r /etc bigred@120.96.143.147:/home/bigred/tt
/# exit
```
參數解說 :   
1. 因為要用scp所以要先下載openssh。  
2. 要將container裡的/etc目錄scp到120.96.143.147:/home/bigred/tt目錄中  

step 2 :
```
pv-hw.yaml內容:

kind: PersistentVolume
apiVersion: v1
metadata:
  name: pv-hw
spec:
  capacity:
    storage: 3Gi
  accessModes:
    - ReadWriteOnce
  local:
    path: "/home/bigred/tt/etc"
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: kubernetes.io/hostname
          operator: In
          values:
          - 120-96-143-147-m1
```
參數解說 :  
1. kind: 宣告產生pv
2. storage: 限制pv的使用容量 (這邊限制3G)  
3. local: path: "/home/bigred/tt/etc" > 永存的項目是目錄區  
4. 並指定 120-96-143-147-m1 這台node至少要有3G空間可以給pv使用  

```
pvc-hw.yaml內容:

kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: pvc-hw
spec:
  accessModes:
    - ReadWriteOnce  
  resources:
    requests:
      storage: 3Gi
```
參數解說 :  
1. kind: 宣告產生pvc  
2. storage: 限制pv的使用容量  
3. 建立 PVC 會立即搜尋可用的 PV，然後建立連接。(這邊的pvc會去找儲存空間至少有3G的pv)  

```
pod-pvc-hw.yaml內容:

kind: Pod
apiVersion: v1
metadata:
  name: pod-pvc-hw
spec:
  volumes:
    - name: pv-etc
      persistentVolumeClaim:
       claimName: pvc-hw
    - name: pv-home
      hostPath:
        path: /home/bigred/tt/home
  containers:
    - name: pod-pvc-hw
      image: quay.io/cloudwalker/alpine
      imagePullPolicy: Always
      stdin: true
      tty: true
      volumeMounts:
        - mountPath: "/etc"
          name: pv-etc
        - mountPath: "/home"
          name: pv-home
```
參數解說 :  
1. kind: 宣告產生pod  
2. volumes: 裡面有 2 個 mount 的路徑  
pv-etc : 會去找 pvc-hw 這個 pvc 要目錄  
pv-home: 執行這個ymal檔的node之/home/bigred/tt/home掛載進來做使用(如果沒有這個目錄會幫忙產生，因為是 hostPath)  
3. container:  
volumeMounts:  會去找這個 pod 的 volume 去使用，pv-etc mount container 裡的 /etc ，pv-home mount container 裡的 /home  

step 3 : 
```
$ kubectl create -f pv-hw.yaml
$ kubectl create -f pvc-hw.yaml
$ kubectl create -f pod-pvc-hw.yaml

$ kubectl exec -it pod-pvc-hw -- sh
/# adduser -s /bin/sh -h /home/bigred -D bigred
/# passwd bigred (接著輸入兩次密碼)
/# exit

$ cat /home/bigred/tt/etc/passwd
你會發現多一行bigred，表示mount成功
$ ls -al /home/bigred/tt/home
你會發現有bigred，表示也mount成功
基本上pv是跟pod無關的，所以你刪掉pod，pv還在
在create一次pod-pvc-hw.yaml也一定會在mount進去

$ kubectl exec -it pod-pvc-hw -- sh
/# su bigred
/$ whoami
bigred
```



## pv 與 pvc 的感情糾葛  
```
$ kubectl delete pvc pvc-local
persistentvolumeclaim "pvc-local" deleted

$ kubectl get pv
NAME       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS     CLAIM               STORAGECLASS   REASON   AGE
pv-local   10Gi       RWO            Retain           Released   default/pvc-local                           7m52s
```
刪掉 pvc 後 原本配給他的 pv 的狀態會變為 Released ，但實際上他現在還無法接受新的 pvc ，必須修改此 pv 裡的記憶(log)，方法如下 :  
```
$ export KUBE_EDITOR="nano"
$ kubectl edit pv pv-local   
.......
  capacity:
    storage: 10Gi
需刪掉的記憶_____________________
  claimRef:
    apiVersion: v1
    kind: PersistentVolumeClaim
    name: pvc-local
    namespace: default
    resourceVersion: "2335811"
    uid: f1e0e01e-890c-11e9-9065-0007324d1e94
需刪掉的記憶_____________________
  local:
    path: /opt/local
.........
```
此時這個 pv 就能配給新的適合的 pvc 囉!!  

---

## :heavy_plus_sign: Deployment  
```
kind: Deployment
apiVersion: apps/v1
metadata:
  name: s1.dep
spec:
  replicas: 2
  selector:
    matchLabels:
      app: s1.pod
  template:
    metadata:
      labels:
        app: s1.pod
    spec:
      containers:
      - name: derbyapp
        image: quay.io/cloudwalker/alpine.derby
        imagePullPolicy: Always
        ports:
        - name: http
          containerPort: 8888 
```
參數解說 :  
Deployment 一定會放在master主機  
立即產生兩個都貼有 app: s1.pod 標籤的pod(因為replicas: 2)  
template : 宣告的是 pod 的設定  
ports : 宣告的協定 http，port號 8888  
replicaset.app : 是 controller 幫忙控制管理 pod  
(\$ kubectl get all 可以看到)  

:warning: 若要刪除整個deployment就直接把這個ymal檔刪除即可，如果只刪除pod會一直重生。  

## :baby_chick: Service  
### 沒指定 type 的 service  
```
kind: Service
apiVersion: v1
metadata:
  name: s1-service
spec:
  selector:
    app: s1.pod
  ports:
    - port: 9999
      targetPort: 8888
```
參數解說 :  
port 號 9999 為 service 開的 port 號  
targetPort 號 8888 為 pod(container) 開的 port 號  

### Nodeport service  

![](https://i.imgur.com/v8jcYbB.png)

```
apiVersion: v1
kind: Service
metadata:  
  name: my-nodeport-service
spec:
  selector:
    app: s1.pod
  type: NodePort
  ports:
  - port: 9999
    targetPort: 8888
    nodePort: 32000
    protocol: TCP 
```
參數解說 :  
nodePort 範圍要在 30000-32767 之間  
port 號 9999 為 service 開的 port 號  
targetPort 號 8888 為 pod(container) 開的 port 號  

### External IP service  
```
kind: Service
apiVersion: v1
metadata:
  name: myextip
spec:
  externalIPs:
  - $IP
  selector:
    app: s1.pod
  ports:
  - port: 8080
    targetPort: 8888
```
參數解說 :  
port : service 跟 node 都開 8080  
targetPort 號 8888 為 pod(container) 開的 port 號  

### DNS for Service  

![](https://i.imgur.com/Lb1rf01.png)

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: k8s-nginx
spec:
  selector:
    matchLabels:
      run: k8s-nginx
  replicas: 3
  template:
    metadata:
      labels:
        run: k8s-nginx
    spec:
      containers:
      - name: k8s-nginx
        image: quay.io/cloudwalker/nginx
        imagePullPolicy: Never
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: svc-cluster
spec:
  selector:
    run: k8s-nginx
  ports:
  - name: http
    port: 80
    protocol: TCP
---
kind: Service
apiVersion: v1
metadata:
  name: svc-headless
spec:
  selector:
    run: k8s-nginx
  ports:
    - name: http
      protocol: TCP
      port: 80
      targetPort: 80
  clusterIP: None
---
```
參數解說 :   
一個 yaml 檔可以宣告產生多個 component，中間用 \-\-\- 分隔。  
clusterIP : None，這個 service 說他不要 cluster ip ，稱為 headless  
**每個 service 都會到 kube-dns 回報自己的名字和 IP， headless 因為沒有自己的 IP，所以他會回報他已圈起的pod的 IP**，實作結果如下:  
```
$ nslookup
> server 10.96.0.10
Default server: 10.96.0.10
Address: 10.96.0.10#53
> svc-headless.default.svc.cluster.local
Server:		10.96.0.10
Address:	10.96.0.10#53

Name:	svc-headless.default.svc.cluster.local
Address: 10.244.1.3
Name:	svc-headless.default.svc.cluster.local
Address: 10.244.2.3
Name:	svc-headless.default.svc.cluster.local
Address: 10.244.1.4

> svc-cluster.default.svc.cluster.local
Server:		10.96.0.10
Address:	10.96.0.10#53

Name:	svc-cluster.default.svc.cluster.local
Address: 10.102.5.155
> exit
```


---

## :adult: K8S 帳號管理憑證  
```
$ mkdir bigboss; cd bigboss
 
$ openssl genrsa -out bigboss.key 2048
Generating RSA private key, 2048 bit long modulus (2 primes)
................................+++++
.....................................................................................+++++
e is 65537 (0x010001)

$ openssl req -new -key ~/bigboss/bigboss.key -out ~/bigboss/bigboss.csr -subj "/CN=bigboss/O=DT"

$ head -n 3 bigboss.csr 
-----BEGIN CERTIFICATE REQUEST-----
MIICZDCCAUwCAQAwHzEOMAwGA1UEAwwFbXVsYW4xDTALBgNVBAoMBGFybXkwggEi
MA0GCSqGSIb3DQEBAQUAA4IBDwAwggEKAoIBAQDorM3IDlgRzbpccPoH+XRokT3a

$ export BASE64_CSR=$(cat bigboss.csr | base64 | tr -d '\n')
```
參數解說 :  
1. 創立使用者的指定資料夾  
2. 產生私鑰 (2048 bytes)名字就叫做bigboss.key  
3. 產生憑證 (內含public key和內有(名/組織/公鑰))  
4. 轉碼:把檔案內容變成64個字表示，憑證檔透過網路傳輸時不會因為有特殊的字元出錯  

```
$ echo 'apiVersion: certificates.k8s.io/v1beta1
kind: CertificateSigningRequest
metadata:
  name: bigboss-csr
spec:
  groups:
  - system:authenticated
  request: ${BASE64_CSR}
#  signerName: kubernetes.io/kube-apiserver-client
  usages:
  - digital signature
  - key encipherment
  - server auth
  - client auth  ' > bigboss-csr.yaml 

$ cat bigboss-csr.yaml | envsubst | kubectl apply -f -
Warning: certificates.k8s.io/v1beta1 CertificateSigningRequest is deprecated in v1.19+, unavailable in v1.22+; use certificates.k8s.io/v1 CertificateSigningRequest
certificatesigningrequest.certificates.k8s.io/bigboss-csr created

$ kubectl get csr
NAME          AGE     SIGNERNAME                   REQUESTOR         CONDITION
bigboss-csr   53s   kubernetes.io/legacy-unknown   kubernetes-admin   Pending

$ kubectl certificate approve bigboss-csr
certificatesigningrequest.certificates.k8s.io/bigboss-csr approved

$ kubectl get csr
NAME           AGE   SIGNERNAME                     REQUESTOR         CONDITION
bigboss-csr   106s   kubernetes.io/legacy-unknown   kubernetes-admin   Approved,Issued
```
參數解說 : 
1. kind: CertificateSigningRequest(種類: 憑證簽核需求)把憑證送到k8s憑證中心做審核
2. usages內容:數位簽章、鑰匙加密解密、Server審核認證、Client審核認證

:warning: 被系統警告當k8s變1.22版之後的版本就不支援，可能是宣告上的語法會不一樣，所以後續要找這個檔案的接替檔  
3. certificate approve: 自己核准憑證  

```
$ kubectl get csr bigboss-csr -o jsonpath='{.status.certificate}' | base64 --decode > /home/bigred/bigboss/bigboss.crt

$ dir ~/bigboss/
.......
-rw-rw-r-- 1 bigred bigred 1.2K Oct 21 21:33 bigred.crt
-rw-rw-r-- 1 bigred bigred  903 Oct 21 21:28 bigred.csr
-rw------- 1 bigred bigred 1.7K Oct 21 21:23 bigred.key

$ sudo kubectl config set-credentials bigboss --client-certificate=/home/bigred/bigboss/bigboss.crt --client-key=/home/bigred/bigboss/bigboss.key
User "bigboss" set.
```
參數解說 : 
1. 把k8s准許後的憑證拿回來
-o: output  
jsonpath=‘{.status.certificate}‘這段是我指定的部分(憑證)就好   
Base64 --decode: 轉換回原本的憑證檔(含有特殊符號)  
2. 被別人蓋章過，並拿回來的憑證(內有公鑰): --client-certificate=/home/bigred/bigboss/bigboss.crt  
自己的私鑰: --client-key=/home/bigred/bigboss/bigboss.key  

:star2: **K8s使用者要在k8s裡面run，
必備: 經核准過後的憑證 & 個人私鑰**
```
$ kubectl config view | grep -A 10 users:
- name: bigboss
  user:
    client-certificate: /home/bigred/bigboss/bigboss.crt
    client-key: /home/bigred/bigboss/bigboss.key
- name: kubernetes-admin
  user:
    client-certificate-data: REDACTED
    client-key-data: REDACTED
```
參數解說 :  
name: kubernetes-admin 這是 K8S 叢集內建的管理者帳號，Linux 的 bigred 使用的是 admin 帳號操作 k8s  

:warning: client-certificate-data: REDACTED 代表憑證資料直接存在 config 物件中，而不是存在外部憑證檔，參考以下步驟解說:  
\$ cat ~/.kube/config   

1. 產生管理憑證  
2. \$ kubectl config view   
```
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: DATA+OMITTED
    server: https://192.168.1.113:6443
  name: kubernetes
contexts:
- context:
    cluster: kubernetes
    user: kubernetes-admin
  name: kubernetes-admin@kubernetes
current-context: kubernetes-admin@kubernetes
kind: Config
preferences: {}
users:
- name: kubernetes-admin
  user:
    client-certificate-data: REDACTED
    client-key-data: REDACTED
```
3. current-context: 得知從哪個入口登入k8s叢集  
根據入口名對應到 name: k8s-admin@k8s  
(a) 使用者的確定  
user: k8s-admin 這個帳號進入 k8s  
(b) 會對應到下面的 users 資訊:  
去看他的憑證有無被蓋過章&是否有私鑰>>確認此使用者是否合法   
4. cluster: k8s  
會對應到上面 clusters 資訊:  
裡面的 server 是 m1 IP, 意思就是透過 m1 連線 k8s  

---

## :mag_right: RBAC (Role-based access control)  
#### 專門授權這個使用者在k8s可以使用甚麼資源(ex. Pods, Deployments, Services, Nodes, and PersistentVolumes)做甚麼事(ex. get, watch, create, delete)  

```
$ kubectl create namespace finance  
$ kubectl get ns  
NAME              STATUS   AGE  
default           Active   21h  
finance           Active   3s  
kube-node-lease   Active   21h  
kube-public       Active   21h  
kube-system       Active   21h  

$ kubectl auth can-i list pods --namespace finance  
yes (admin有所有執行權限)   
$ kubectl auth can-i list pods --namespace finance --as bigboss  
no  
```

### role 的 yaml 檔

```
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  namespace: finance   
  name: finance-reader
rules:
- apiGroups: [""] # "" indicates the core API group
  resources: ["pods", "services"]
  verbs: ["get", "watch", "list"] '
```
參數解說:
1. Role: 針對component宣告可以執行甚麼樣的動作
2. rules: 人/可以用什麼資源/可以下什麼指令

:arrow_right: 也可以套用至整個k8s叢集，kind宣告內容改為 kind: ClusterRole

### RoleBinding 的 yaml 檔  
```
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: finance-read-access-pondai
  namespace: finance 
subjects:
- kind: User
  name: bigboss
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role #this must be Role or ClusterRole
  name: finance-reader 
  apiGroup: rbac.authorization.k8s.io
```
參數解說:  
1. Rolebinding: 針對使用者宣告誰可以做這件事  
2. 允許這個使用者(bigboss)遵守這個role(finance-reader)  

:arrow_right: 也可以套用至整個k8s叢集，kind宣告內容改為 kind: ClusterRoleBinding

## :bookmark_tabs: K8s contexts
#### 根據 context 設定選擇，決定你 default 使用哪一個叢集，在哪一個 namespace 作業，使用哪一個身分，這樣可以減少我們所需要打的指令

```
$ kubectl config get-contexts
CURRENT   NAME      CLUSTER   AUTHINFO   NAMESPACE
*         default   default   default 

$ kubectl config set-context finance-context --cluster=kubernetes --namespace=finance --user=bigboss

$ kubectl config get-contexts
CURRENT   NAME              CLUSTER      AUTHINFO   NAMESPACE
*         default           default      default    
          finance-context   kubernetes   bigboss    finance 

$ kubectl config use-context finance-context

$ kubectl config get-contexts
CURRENT   NAME              CLUSTER      AUTHINFO   NAMESPACE
          default           default      default    
*         finance-context   kubernetes   bigboss    finance 

#透過kubectl config use-context，這邊的 * 就會顯示當前使用在哪個context
```

## 遠端操作 kubernetes 叢集  
**k8s 作法:**  
```
$ curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
#上網下載kubectl的功能
$ sudo chmod +x kubectl && sudo mv kubectl /usr/bin

$ scp -r bigred@120.96.143.157:~/.kube .
#沒有這個設定檔就無法操作kubectl這個指令

$ scp -r bigred@120.96.143.157:~/bigboss .
(如果要使用 bigboss 使用者，還需要 bigboss 的憑證和私鑰)
```
:warning: 若不想讓其他使用者使用admin權限，可以在scp過去之後先把 .kube/config 檔的admin重要資訊刪除

**k3s 作法:**  
```
$ curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"

$ sudo chmod +x kubectl && sudo mv kubectl /usr/bin

$ mkdir .kube

$ scp -r bigred@192.168.25.41:/etc/rancher/k3s/k3s.yaml .kube

$ sudo nano .kube/k3s.yaml 
server: https://127.0.0.1:6443 改成 server: https://192.168.25.41:6443

$ mv .kube/k3s.yaml .kube/config

```








