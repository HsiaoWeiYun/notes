### CentOS 7 MySQL5.7 安裝

1. 下載MySQL
`sudo rpm -Uvh https://dev.mysql.com/get/mysql57-community-release-el7-11.noarch.rpm`

2. 安裝MySQL
`sudo yum install mysql-community-server`

3. 設定開機自動啟動
`sudo systemctl enable mysqld`

4. 啟動MySQL
`sudo systemctl start mysqld`

5. 找出隨機密碼
`sudo grep 'temporary password' /var/log/mysqld.log`

6. 執行MySQL安全性工具
`sudo mysql_secure_installation`
