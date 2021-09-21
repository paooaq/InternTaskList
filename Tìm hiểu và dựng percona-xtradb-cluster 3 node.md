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

