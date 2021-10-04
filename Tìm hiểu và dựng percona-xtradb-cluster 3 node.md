# Tìm hiểu và dựng percona-xtradb-cluster 3 node

1. ## Percona Xtradb Cluster là gì

   Percona XtraDB Cluster là một giải pháp mã nguồn mở hoàn toàn và có tính khả dụng cao cho MySQL. Nó tích hợp Percona Server và Percona XtraBackup với thư viện Galera cho phép sao chép đa nguồn đồng bộ.

   Một cluster bao gồm các node, trong đó mỗi node chứa cùng một tập dữ liệu được đồng bộ hóa giữa các node. Cấu hình được đề xuất là có ít nhất 3 node, nhưng bạn cũng có thể có 2 node. Mỗi node là một phiên bản MySQL Server thông thường (ví dụ: Percona Server). Bạn có thể chuyển đổi một phiên bản MySQL Server hiện có thành một node và chạy cluster bằng cách sử dụng node này làm cơ sở dữ liệu. Ngoài ra, bạn cũng có thể tách bất kỳ node nào ra khỏi cluster và sử dụng nó như một phiên bản MySQL Server thông thường.

   ![image-20210921002204240](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20210921002204240.png)

2. ## Cài Đặt Và Cấu Hình Percona Xtradb Cluster 8.0 Trên ubuntu 20.04

   1. #### Chuẩn bị

      - Để cài đặt Percona XtraDB Cluster, đầu tiên chúng ta cần chuẩn bị 3 máy chủ ubuntu 20.04

        |  node  |       IP       |
        | :----: | :------------: |
        | node-1 | 192.168.30.192 |
        | node-2 | 192.168.30.136 |
        | node-3 | 192.168.30.174 |

        

      - allow firewall các port bên dưới:

        - 3306
        - 4444
        - 4567
        - 4568

   2. #### Cấu hình kho lưu trữ Percona

      - ```
        sudo apt-get update
        sudo apt-get install -y wget gnupg2 curl lsb-release 
        ```

      - Fetch the repository package: 

        ```
        wget https://repo.percona.com/apt/percona-release_latest.generic_all.deb
        ```

      - Install the downloaded repository package with `dpkg`:

        ```
        sudo dpkg -i percona-release_latest.generic_all.deb
        ```

      - Check the repository setup for the Percona original release list in the `/etc/apt/sources.list.d/percona-original-release.list` file

      - Refresh the local cache to update the package information:

        ```
        sudo apt-get update
        ```
        
      - enable-only PXC-80 release
      
        ```
        sudo percona-release enable-only pxc-80 experimental
        sudo percona-release enable tools release
        ```
      
      - install percona extradb cluster :
      
        ```
        sudo apt update
        sudo apt-get install percona-xtradb-cluster
        ```
      
   3. #### Cấu hình động bộ giữa các node
   
      - Dừng dịch vụ mysql trước khi cấu hình
   
        ```
        systemctl stop mysql
        ```
   
      - Cấu hình mỗi server (/etc/mysql/mysql.conf.d/mysqld.cnf)
   
        1. Cấu hình node 1:
   
           ```
           [mysqld]
           
           datadir=/var/lib/mysql
           user=mysql
           
           # Path to Galera library
           wsrep_provider=/usr/lib/libgalera_smm.so
           
           # Cluster connection URL contains the IPs of node#1, node#2 and node#3
           wsrep_cluster_address=gcomm://192.168.30.192,192.168.30.136,192.168.30.174
           
           # In order for Galera to work correctly binlog format should be ROW
           binlog_format=ROW
           
           # MyISAM storage engine has only experimental support
           default_storage_engine=InnoDB
           
           # This changes how InnoDB autoincrement locks are managed and is a requirement for Galera
           innodb_autoinc_lock_mode=2
           
           # Node #1 address
           wsrep_node_address=192.168.30.192
           
           # SST method
           wsrep_sst_method=xtrabackup-v2
           
           # Cluster name
           wsrep_cluster_name=my_ubuntu_cluster
           
           # Authentication for SST method
           wsrep_sst_auth="sstuser:s3cretPass"
           ```
   
        2. Cấu hình node 2:
   
           ```
           [mysqld]
           
           datadir=/var/lib/mysql
           user=mysql
           
           # Path to Galera library
           wsrep_provider=/usr/lib/libgalera_smm.so
           
           # Cluster connection URL contains IPs of node#1, node#2 and node#3
           wsrep_cluster_address=gcomm://192.168.30.192,192.168.30.136,192.168.30.174
           
           # In order for Galera to work correctly binlog format should be ROW
           binlog_format=ROW
           
           # MyISAM storage engine has only experimental support
           default_storage_engine=InnoDB
           
           # This changes how InnoDB autoincrement locks are managed and is a requirement for Galera
           innodb_autoinc_lock_mode=2
           
           # Node #2 address
           wsrep_node_address=192.168.30.136
           
           # Cluster name
           wsrep_cluster_name=my_ubuntu_cluster
           
           # SST method
           wsrep_sst_method=xtrabackup-v2
           
           #Authentication for SST method
           wsrep_sst_auth="sstuser:s3cretPass"
           ```
   
        3. Cấu hình node 3:
   
           ```
           [mysqld]
           
           datadir=/var/lib/mysql
           user=mysql
           
           # Path to Galera library
           wsrep_provider=/usr/lib/libgalera_smm.so
           
           # Cluster connection URL contains IPs of node#1, node#2 and node#3
           wsrep_cluster_address=gcomm://192.168.30.192,192.168.30.136,192.168.30.174
           
           # In order for Galera to work correctly binlog format should be ROW
           binlog_format=ROW
           
           # MyISAM storage engine has only experimental support
           default_storage_engine=InnoDB
           
           # This changes how InnoDB autoincrement locks are managed and is a requirement for Galera
           innodb_autoinc_lock_mode=2
           
           # Node #3 address
           wsrep_node_address=192.168.30.174
           
           # Cluster name
           wsrep_cluster_name=my_ubuntu_cluster
           
           # SST method
           wsrep_sst_method=xtrabackup-v2
           
           #Authentication for SST method
           wsrep_sst_auth="sstuser:s3cretPass"
           ```
   
   4. #### Khởi động dịch vụ mysql trên từng node
   
      - node 1
   
        ```
        systemctl start mysqld@bootstrap.service
        systemctl stop mysql@bootstrap.service
        systemctl start mysql
        ```
   
      - node 2&3
   
        ```
        systemctl start mysql
        ```
   
   5. #### Kiểm tra tính năng đồng bộ
   
      - Tạo database mới trên node 2:
   
        ```
        mysql@pxc2> CREATE DATABASE percona;
        Query OK, 1 row affected (0.01 sec)
        ```
   
      - Tạo bảng example trên node thứ 3:
   
        ```
        mysql@pxc3> USE percona;
        Database changed
        
        mysql@pxc3> CREATE TABLE example (node_id INT PRIMARY KEY, node_name VARCHAR(30));
        Query OK, 0 rows affected (0.05 sec)
        ```
   
      - Chèn bản ghi trên node 1:
   
        ```
        mysql@pxc1> INSERT INTO percona.example VALUES (1, 'percona1');
        Query OK, 1 row affected (0.02 sec)
        ```
   
      - Truy xuất tất cả hàng trên node 2:
   
        ```
        mysql@pxc2> SELECT * FROM percona.example;
        +---------+-----------+
        | node_id | node_name |
        +---------+-----------+
        |       1 | percona1  |
        +---------+-----------+
        1 row in set (0.00 sec)
        ```
      
   6. #### Kiểm tra tính năng Recovery
   
      1. Một nút dừng lại 1 cách có mục đích (ví dụ node 2 )
   
         ![image-20211004101333830](D:\OneDrive\FPT\tasklist\image\image-20211004101333830.png)
   
         Trạng thái khi xem từ node 1khi tắt và bật node 2
      
         Khi node 2 khởi động lại thì nó tự tham gia lại vào cụm dựa trên biến **wsrep_cluster_address**
      
      2. Hai nút dừng lại 1 cách có mục đích ( node 2 và node 3)
      
         ![image-20211004103758763](D:\OneDrive\FPT\tasklist\image\image-20211004103758763.png)
      
         Trạng thái trên node 1 và thêm dữ liệu vào database
      
         ![image-20211004104005707](D:\OneDrive\FPT\tasklist\image\image-20211004104005707.png)
      
         Khởi động node 2 kiểm tra cluster_size và truy xuất dữ liệu
      
         ![image-20211004104212017](D:\OneDrive\FPT\tasklist\image\image-20211004104212017.png)
      
         Khởi động node 3 kiểm tra cluster size và truy xuất dữ liệu
      
      3. Cả 3 node đều dừng 1 cách có mục đích
      
         Khi đó mình sẽ so sánh chỉ số seqno trong tệp /var/lib/mysql/grastate.dat, node nào có chỉ số cao nhất thì node đó khả năng cao nhất là node cuối cùng
      
         ![image-20211004152421067](D:\OneDrive\FPT\tasklist\image\image-20211004152421067.png)
      
         khi đó mình khởi động node đầu tiên trên node có chỉ số seqno cao nhất qua lệnh 
      
         ```
         systemctl start mysql@bootstrap.service
         ```
      
      4. Cả 3 node dừng một cách bất ngờ
      
         Tình huống này có thể xảy ra trong trường hợp mất điện trung tâm dữ liệu hoặc khi gặp lỗi MySQL hoặc Galera. Ngoài ra, nó có thể xảy ra do tính nhất quán của dữ liệu bị xâm phạm trong đó cụm phát hiện rằng mỗi nút có dữ liệu khác nhau. Các `grastate.dat`tập tin không được cập nhật và không chứa một số thứ tự hợp lệ (seqno). 
      
         Tìm node `safe_to_bootstrap: 1` và khởi động node đó thành node đầu tiên
      
         ![image-20211004153014440](D:\OneDrive\FPT\tasklist\image\image-20211004153014440.png)
      
         

## Tài liệu tham khảo

https://www.percona.com/doc/percona-xtradb-cluster/5.6/howtos/ubuntu_howto.html

https://blog.vinahost.vn/cau-hinh-percona-xtradb-cluster-8-0-tren-centos-7
