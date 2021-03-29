---
title: Laradock Mysql 連接問題
date: 2021-03-29
categories:
- Laravel
- Laradock
- error
tags:
- Laravel
- Laradock
- error
---



# Laradock Mysql 連接問題
###### tags:  `Laravel` `Laradock` `error`    

# 在這紀錄一下問題點，以防下次發生

laravel的.env檔配置

```
DB_CONNECTION=mysql
DB_HOST=mysql
DB_PORT=3306 #這裡要設定為3306，不是3308
DB_DATABASE=action_30_day
DB_USERNAME=root
DB_PASSWORD=root
```

會顯示以下錯誤
```
SQLSTATE[HY000] [2002] No such file or directory 
```

將 DB_HOST 設定成：
```
DB_HOST=127.0.0.1
```

錯誤反而會變成

```
 SQLSTATE[HY000] [2002] Connection refused
```

正確要改成
```
DB_HOST=mysql
DB_PORT=3306 #這裡要設定為3306，不是3308
```

參考網址 : https://learnku.com/articles/5893/laradock-database-connection-problem
