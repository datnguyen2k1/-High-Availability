###High-Availability

I. Xây dựng HAProxy với 2 Server chạy WordPress

  https://github.com/datnguyen2k1/HAProxy

II. Xây dựng HAProxy với 2 Server chạy SQL có sử dụng Master-Slave

  2.1 Chuẩn bị:
   -HAProxy Server : 172.20.227.227
   - MySQL Master: 	172.20.228.186
   - MySQL Slave: 172.20.228.189
  2.3 Cài đặt HAProxy Server:
   
   - Cài đặt HAProxy:
    
         apt-get install haproxy -y

   - Cấu hình file haproxy.cfg
        
    global
        log 127.0.0.1 local0 notice
        user haproxy
        group haproxy

    defaults
        log global
        retries 2
        timeout connect 3000
        timeout server 5000
        timeout client 5000

    listen mysql-cluster
        bind <ip_haproxy>:3306
        mode tcp
        option tcpka
        balance roundrobin
        server mysql-1 <ip_master>:3306 check
        server mysql-2 <ip_slave>:3306 check

   - Cấu hình lại localhost để file haproxy.cfg request về HTTP
       
       
       
    <ip_master> <tên_server1>
    <ip_slave> <tên_server2>
  
  2.4 Cài đặt với SQL master và SQL slave:
  
  - Cài đặt MySQL:
  
        apt install mysql-server -y
  
  -Thực hiện mở port 3306 và mở cmt server_id, log và bind_address trong mysqld.conf( master: server_id=1, slave: server_id=2, bind_address=0.0.0.0):
  
    sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf
    
  - Tạo user cho phép HA server remote:
      
          root@sql_master: mysql -urooot "Insert into mysql.user (Host,User,ssl_cipher,x509_issuer,x509_subject) values ('<ip_haproxy>','<user>','abc','abc','abc');"
  
  - Cấp quyền:
          
           root@sql_master: mysql -urooot "GRANT ALL PRIVILEGES ON *.* TO '<user>'@'<ip_haproxy>';"
           
   - Đặt mật khẩu:
  
           root@sql_master: mysql -urooot "GRANT ALL PRIVILEGES ON *.* TO '<user>'@'<ip_haproxy>'; FLUSH PRIVILEGES;"
           
   2.5 Tiến hành Replication dữ liệu SQL master sang SQL slave:
   
   - Tạo người dùng để Repli:
   
         root@sql_master: mysql -urooot "CREATE USER 'repl'@'<ip_slave>' IDENTIFIED WITH mysql_native_password BY '123456';"
        
   - Cấp quyền Read:
   
         root@sql_master: mysql -urooot "GRANT REPLICATION SLAVE ON *.* TO 'repl'@'<ip_slave>';"
         
   - Sang SQL slave và thực hiện Replication với SQL master:
   
          root@sql_slave: mysql -urooot "STOP SLAVE; CHANGE MASTER TO MASTER_HOST='<ip_master>', MASTER_USER='<user>', MASTER_PASSWORD='123456', MASTER_PORT=3306, MASTER_LOG_FILE='<ten_file_log>', MASTER_LOG_POS=<vi_tri_file>; START SLAVE;"
          
   - Check kết nối, nếu Master_Host là ip SQL master là thành công hoặc sang master tạo database rồi qua slave check có là được:
    
          root@sql_slave: mysql -urooot "SHOW SLAVE STATUS\G"
   2.6 Thực hiện cài WordPress cho 2 Server được link HAProxy trước đó:
   
   - Sau khi thực hiện các bước cài WordPress thì file wp-config.php cấu hình như sau: db_name là tên db của master, user_name là user đc tạo cho haproxy để kết nối với 2 server SQL, mật khẩu là của user đó luôn, còn về phần host là lấy của HAProxy link 2 server SQL
   
   - Kết quả thực hiện là lấy IP của HAProxy link 2 server WP chạy:
    ![image](https://user-images.githubusercontent.com/74607192/156726131-98ea9618-e3f0-4296-b9f9-573867b973a4.png)
    
   - Lưu ý: chúng ta có thể làm ngược lại Relication đối với 2 server SQL để nâng cao độ chịu lỗi của hệ thống.
   
III. Liên kết hệ thống lại với nhau 
![image](https://user-images.githubusercontent.com/74607192/156492905-04652087-f408-49bc-870b-5f1b52bf4c9d.png)
