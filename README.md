# sysbench_stress_test_Note

### 參考網址: https://www.netadmin.com.tw/netadmin/zh-tw/technology/A8DF64E5075C4DC6991FCD37AA2E1173
### 官方安裝步驟: https://github.com/akopytov/sysbench
sysbench是開源碼社群中頗富盛名的壓力測試軟體，主要用來測試主機的檔案系統及記憶體的讀寫效能、CPU等系統效能。
除了上述傳統的系統效能測試項目外，sysbench更提供資料庫的運作效能測試，目前提供MySQL、Oracle、PostgreSQL等相關著名的資料庫軟體。
在測試資料庫效能方面，主要重點在於OLTP（On-Line Transaction Processing system，連線交易處理系統）的測試。所謂的線上交易處理（OLTP）指的是在交易（Transaction）進行時，可針對交易資料進行即時處理，而非傳統的批次處理。
所以，sysbench測試OLTP的效能重點是，當使用者在提出一個交易要求，而資料庫系統在接收到該筆交易要求並運算完成之後回覆給使用者的整段時間，sysbench即是藉著測量此段時間的長短來評估資料庫的效能好壞。

sysbench 測試前，先進行MySQL安裝
本次使用Docker進行操作

## 版本
MySQL 8.0
sysbench 1.0.20
## Docker-compose.yml

```
version: "3.1"
services:
  db:
    image: mysql:8.0
    restart: always
    container_name: mysql
    command: --default-authentication-plugin=mysql_native_password 
    hostname: mysql
    environment:
      MYSQL_ROOT_PASSWORD: test
      TZ: Asia/Taipei
    ports:
      - "3306:3306"
    
  adminer:
    image: adminer
    restart: always
    ports:
      - 8080:8080
```


# MySQL
## MySQL官方文件參考 https://dev.mysql.com/doc/refman/8.0/en/upgrading-from-previous-series.html

Docker db的配置中command: **--default-authentication-plugin=mysql_native_password** 這欄設定非常重要，若沒加上默認設定 sysbench 實際在測試時是連結不上MySQL
若使用直接安裝的MySQL則在 vim /etc/my.cnf [mysqld] 加入這行 --default-authentication-plugin=mysql_native_password
mysql的設定大致上就這樣

# Sysbench 
為sysbench創建一個MySQL數據庫和用戶：
```
mysql> CREATE SCHEMA sbtest;
mysql> CREATE USER sbtest@'%' IDENTIFIED BY 'password';
mysql> GRANT ALL PRIVILEGES ON sbtest.* to sbtest@'%';
```

查看版本
```
$ sysbench --version
```

查看幫助
```
$ sysbench --help
```
可以查詢到支援的選項
測試腳本為.lua
sysbench 腳本路徑
/usr/share/sysbench/

sysbench 測試三步驟: 創建數據 prepare 執行測試 run 清除數據 cleanup


```
sysbench \
oltp_read_write.lua \
 --db-driver=mysql \
--table-size=1000000 \
--tables=16 \
--threads=16 \
--mysql-host=127.0.0.1 \
--mysql-port=3306 \
--mysql-user=sbtest \
--mysql-password=password \
prepare 

sysbench \
oltp_read_write.lua \
--db-driver=mysql \
--mysql_storage_engine=inndb \
--table-size=1000000 \
--threads=24 \
--histogram=on \
--time=600 \
--mysql-host=127.0.0.1 \
--mysql-port=3306 \
--mysql-user=sbtest \
--mysql-password=password \
--report-interval=5 \
run 

ysbench oltp_read_write.lua \
--db-driver=mysql \
--report-interval=2  \
--tables=24 \
--threads=24 \
--mysql-host=127.0.0.1 \
--mysql-port=3306 \
--mysql-user=sbtest \
--mysql-password=password \
cleanup
```

