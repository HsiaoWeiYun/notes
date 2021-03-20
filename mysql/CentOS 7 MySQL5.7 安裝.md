### CentOS 7 MySQL5.7 安裝

1. 建立MySQL用戶與群組 <br> 
`sudo groupadd mysql` <br>
`sudo useradd -g mysql mysql -s /sbin/nologin`

2. 下載與解壓MySQL <br>
`cd /usr/local` <br>
`wget https://cdn.mysql.com//Downloads/MySQL-5.7/mysql-5.7.33-linux-glibc2.12-x86_64.tar.gz` <br>
`tar -zxvf mysql-5.7.33-linux-glibc2.12-x86_64.tar.gz`

3.  製作軟連結 <br>
`ln -s mysql-5.7.33-linux-glibc2.12-x86_64 mysql`

4. 建立資料位置
`mkdir -p /data/mysql`

5. 變更資料夾權限 <br>
`chown mysql:mysql -R /usr/local/mysql` <br>
`chown mysql:mysql -R /data/mysql`

6. 編輯/etc/my.cnf <br>
> user=mysql  
port=3306  
basedir=/usr/local/mysql  
datadir=/data/mysql  
socket=/tmp/mysql.sock  
character-set-server=utf8mb4  
server-id=3306100  
sync_binlog=1  
log-bin=/data/mysql/mysql-binlog  
log-error=/data/mysql/error.log  
symbolic-links=0  
!includedir /etc/my.cnf.d

7. 初始化資料庫 <br>
`./mysqld --defaults-file=/etc/my.cnf --basedir=/usr/local/mysql --datadir=/data/mysql --user=mysql --initialize` <br>

8. 找出初始化的密碼  
`cat /data/mysql/error.log | grep password`  

9. 啟動資料庫  
`./mysqld_safe --defaults-file=/etc/my.cnf &` <br>

10. 使用剛剛找出的臨時密碼登入MySQL, 並把密碼改掉 <br>
`mysql -u root -p`<br>
`set password = '你的密碼';`<br>
`alter user 'root'@'localhost' password expire never;`<br>
`flush privileges;`

***
### 啟動時error.log顯示錯誤: error while loading shared libraries: libnuma.so.1: cannot open shared object file: No such file or directory  

解法: `yum -y install numactl`

### Mysql 連線方式 <br>
TCP/IP: `mysql -u <帳號> -p -P <port> -h <IP>` <br>
Socket: `mysql -u <帳號> -p -S <socket文件位置 (設定檔內可指定)>` ex: `./mysql -u root -p -S /tmp/mysql.sock`

### 關閉資料庫  
`./mysqladmin -u root -p shutdown`

