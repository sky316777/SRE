# 判斷使用者code(4/12)
判斷使用者是否存在code


---

#!/bin/bash

for var in $@

do

if [ "$(cat /etc/passwd|grep ^$var)" == "" ]

then

sudo useradd -m -s /bin/bash $var && echo "no 

exist.Help you create one"

else

echo "the user is exist"

fi

done


---
![](https://i.imgur.com/fFAjlBJ.png)
