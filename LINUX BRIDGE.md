# LINUX BRIDGE 

# 1. Tổng quan về Linux Bridge

## 1.1. Giới thiệu

-	**Linux bridge** là một phần mềm được tích hợp vào trong nhân Linux để giải quyết vấn đề ảo hóa phần network trong các máy vật lý. Về mặt logic Linux bridge sẽ tạo ra một con switch ảo để cho các VM kết nối được vào và có thể nói chuyện được với nhau cũng như sử dụng để ra mạng ngoài

-	Bản chất, linux bridge sẽ tạo ra các switch layer 2 kết nối các máy ảo (VM) để các VM đó giao tiếp được với nhau và có thể kết nối được ra mạng ngoài. Linux bridge thường sử dụng kết hợp với hệ thống ảo hóa KVM-QEMU.

-	Linux Bridge thật ra chính là một switch ảo và được sử dụng với ảo hóa KVM/QEMU. ***Nó là 1 module trong nhân kernel***. Sử dụng câu lệnh `brctl` để quản lý .

-	Mô tả linux bridge (trường hợp cơ bản nhất):

<img src = "http://imgur.com/LpMlNof.jpg">

***Lưu ý***: Linux bridge được dùng kết hợp với các card ethernet của máy host. Không sử dụng với card wireless. (phần này tìm hiểu sau). 

## 1.2. Cấu trúc hệ thống sử dụng  Linux bridge

<img src = "http://imgur.com/7d8bY6u.jpg">

Khái niệm về physical port và virtual port:

\- **Virtual Computing Device**: Thường được biết đến như là máy ảo VM chạy trong host server 

\- **Virtual NIC (vNIC)**: máy ảo VM có virtual network adapters(vNIC) mà đóng vai trò là NIC cho máy ảo.

\- **Physical swtich port**: Là port sử dụng cho Ethernet switch, cổng vật lý xác định bởi các port RJ45. Một port RJ45 kết nối tới port trên NIC của máy host.

\- **Virtual swtich port**: là port ảo tồn tại trên virtual switch. Cả virtual NIC (vNIC) và virtual port đều là phần mềm, nó liên kết với virtual cable kết nối vNIC

# 2. Tìm hiểu Linux Bridge

## 2.1. Cấu trúc và các thành phần

Cấu trúc Linux Bridge:

<img src = "http://imgur.com/J0ZvKPk.jpg">

Một số khái niệm liên quan tới linux bridge:

\-	**Port**: tương đương với port của switch thật

\-	**Bridge**: tương đương với switch layer 2

\-	**Tap**: hay tap interface có thể hiểu là giao diện mạng để các VM kết nối với bridge cho linux bridge tạo ra (nó nằm trong nhân kernel, hoạt động ở lớp 2 của mô hình OSI)

\-	**fd**: forward data - chuyển tiếp dữ liệu từ máy ảo tới bridge.

## 2.2. Các tính năng

\- 	**STP**: Spanning Tree Protocol - giao thức chống loop gói tin trong mạng.

\-	**VLAN**: chia switch (do linux bridge tạo ra) thành các mạng LAN ảo, cô lập traffic giữa các VM trên các VLAN khác nhau của cùng một switch.

\-	**FDB**: chuyển tiếp các gói tin theo database để nâng cao hiệu năng switch.

# 3. Một số khái niệm liên quan

## 3.1.	Port

- Trong networking, khái niệm port đại diện cho điểm vào ra của dữ liệu trên máy tính hoặc các thiết bị mạng. Port có thể là khái niệm phần mềm hoặc phần cứng. 
Software port là khái niệm tồn tại trong hệ điều hành. Chúng thường là các điểm vào ra cho các lưu lượng của ứng dụng. Tức là khái niệm port mức logic. Ví dụ: port 80 trên server liên kết với Web server và truyền các lưu lượng HTTP. 

- Hardware port (port khái niệm phần cứng): là các điểm kết nối lưu lượng ở mức khái niệm vật lý trên các thiết bị mạng như switch, router, máy tính, … ví dụ: router với cổng kết nối RJ45 (L2/Ethernet) kết nối tới máy tính của bạn.

- Physical switch port: Thông thường chúng ta hay sử dụng các switch L2/ethernet với các cổng RJ45. Một đầu connector RJ45 kết nối port trên switch tới các port trên NIC của máy tính. 

- Virtual switch port: giống như các physical switch port mà tổn tại như một phần mềm trên switch ảo. cả virtual NIC và virtual port đều duy trì bởi phần mềm, được kết nối với nhau thông qua virtual cable.

## 3.2.	Uplink port

-	Uplink port là khái niệm chỉ điểm vào ra của lưu lượng trong một switch ra các mạng bên ngoài. Nó sẽ là nơi tập trung tất cả các lưu lượng trên switch nếu muốn ra mạng ngoài. 

-	Khái niệm virtual uplink switch port được hiểu có chức năng tương đương, là điểm để các lưu lượng trên các máy guest ảo đi ra ngoài máy host thật, hoặc ra mạng ngoài. Khi thêm một interface trên máy thật vào bridge (tạo mạng bridging với interface máy thật và đi ra ngoài), thì interface trên máy thật chính là virtual uplink port.

## 3.3.	Tap interface

- Ethernet port trên máy ảo VM (mô phỏng pNIC) thường gọi là vNIC (Virtual NIC). Virtual port được mô phỏng với sự hỗ trợ  của KVM/QEMU.

- Port trên máy ảo VM chỉ có thể xử lý các frame Ethernet. Trong môi trường thực tế (không ảo hóa) interface NIC vật lý sẽ nhận và xử lý các khung Ethernet. Nó sẽ bóc lớp header và chuyển tiếp payload (thường là gói tin IP) tới lên cho hệ điều hành. Tuy nhiên, với môi trường ảo hóa, nó sẽ không làm việc vì các virtual NIC sẽ mong đợi các khung Ethernet. 

- Tap interface là một khái niệm về phần mềm được sử dụng để nói với Linux bridge là chuyến tiếp frame Ethernet vào nó. Hay nói cách khác, máy ảo kết nối tới tap interface sẽ có thể nhận được các khung frame Ethernet thô. Và do đó, máy ảo VM có thể tiếp tục được mô phỏng như là một máy vật lý ở trong mạng.

- Nói chung, tap interface là một port trên switch dùng để kết nối với các máy ảo VM.

## 3.4.	Cấu hình linux bridge khởi động cùng hệ điều hành

- Khi tạo bridge bằng command brctl addbr thì bridge là non-persistent bridge (tức là sẽ không có hiệu lực khi hệ thống khởi động lại).

    `brctl addbr br1 `

- Ban đầu, khi mới được tạo, bridge sẽ có một địa chỉ MAC mặc định ban đầu. Khi thêm một NIC của host vào thì MAC của bridge sẽ là MAC của NIC luôn. Khi del hết các NIC của host trên bridge thì MAC của bridge sẽ về là 00:00:00:00:00:00 và chờ khi nào có NIC khác add vào thì sẽ lấy MAC của NIC đó.

- Khi cấu hình bằng câu lệnh `brctl`, các ảnh hưởng của nó sẽ biến mất sau khi khởi động lại hệ thống host server. Để lưu lại thông tin cấu hình trên bridge và khởi động lại cùng hệ thống thì nên lưu lại cấu hình vào file (Ghi vào file, khi boot lại hệ thống, thông tin trong file cũng được cấu hình lại. Những thông tin được lưu dưới dạng file, thì luôn khởi động cùng hệ thống - nên có thể coi là vĩnh viễn - trừ khi tự tay stop lại dịch vụ.)

- Cấu hình trong file /etc/network/interfaces

    ```
    # This file describes the network interfaces available on your system
    # and how to activate them. For more information, see interfaces(5).
    # The loopback network interface
    auto lo 
    iface lo inet loopback
    # Set up interfaces manually, avoiding conflicts with, e.g., network manager
    iface eth0 inet manual
    iface eth1 inet manual
    # Bridge setup
    auto br0
    iface br0 inet static
        bridge_ports eth0 eth1
        address 192.168.1.2         # Địa chỉ của br1 có thể là cùng dải địa chỉ của eth0 hoặc eth1 tùy ý. 
        broadcast 192.168.1.255
        netmask 255.255.255.0
        gateway 192.168.1.1
    ```

    Khi khởi động lại, hệ thông sẽ đọc file cấu hình, và cấp địa chỉ cho interface br0 (đại điện cho bridge br0) thông qua liên kết giữa eth0 và mạng 192.168.1.0/24. Và các máy VM kết nối tới bridge, lấy chung dải mạng với bridge thông qua liên kết uplink qua eth0 và có thể liên lạc với mạng bên ngoài.

- Tham khảo thêm cấu hình các thông số khác cho linux bridge trong file `/etc/network/interfaces`  tại: http://manpages.ubuntu.com/manpages/xenial/man5/bridge-utils-interfaces.5.html

# 4. LAB cơ bản

## 4.1. Chuẩn bị và mô hình

- Topology:


- Chuẩn bị:

    - Một máy tính với  card eth1 thuộc dải 192.168.30.0/24, cài HĐH ubuntu 20.04.

    - Máy tính đã cài đặt sẵn các công cụ để quản lý và tạo máy ảo KVM. 


- Nội dung bài lab: Tạo một switch ảo br1 và gán interface eth1 vào switch đó, tạo một máy ảo bên trong máy host, gắn vào tap interface của switch và kiểm tra địa chỉ được cấp phát. Tạo 2 VM trong host cùng gắn vào tap interface của switch, ping kiểm tra kết nối).

## 4.2. Cấu hình

- **Bước 1:** Tạo switch ảo br1. Nếu đã tồn tại có thể xóa switch này đi và tạo lại:

    `brctl delbr br1 # xóa đi nếu đã tồn tại`

    `brctl addbr br1 # tạo mới`

- **Bước 2:** Gán eth1 vào swicth br1

    `brctl addif br1 eth1`

    `brctl stp br1 on # enable tính năng STP nếu cần`

- **Bước 3:** Khi tạo một switch mới br1, trên máy host sẽ xuất hiện thêm 1 NIC ảo trùng tên switch đó (br1).

    Ta có thể cấu hình xin cấp phát IP cho NIC này sử dụng command hoặc cấu hình trong file /etc/network/interfacesđể giữ cấu hình cho switch ảo sau khi khởi động lại:

    - Cấu hình bằng command:

        `ifconfig eth1 0 # xóa IP của eth1`

        `dhclient br1`

    - Cấu hình trong file `/etc/network/interfaces`: Nếu trước đó trong file /etc/network/interfaces đã cấu hình cho NIC eth1, ta phải comment lại cấu hình đó hoặc xóa cấu hình đó đi và thay bằng các dòng cấu hình sau:

        ```
        auto br1
        iface br1 inet dhcp
        bridge_ports eth1
        bridge_stp on
        bridge_fd 0
        bridge_maxwait 0
        ```

- **Bước 4**: Khởi động lại các card mạng và kiểm tra lại cấu hình bridge:

    `ifdown -a && ifup -a # khởi động lại tất cả các NIC`

    `brctl show # kiểm tra cấu hình switch ảo`

    Kết quả kiểm tra cấu hình sẽ tương tự như sau:

    ```
    Bridge	name		bridge id	STP	enabled	interfaces
    br0		8000.000		c29586f24	yes	eth0
    br1		8000.000		c29586f2e	yes     eth1
    ```

    Kết quả cấu hình thành công gắn NIC eth1 vào switch ảo br1 sẽ hiển thị như đoạn mã trên.


# 5. Tham khảo

[1] - http://manpages.ubuntu.com/manpages/trusty/man5/bridge-utils-interfaces.5.html

[2] - https://github.com/thaihust/Thuc-tap-thang-03-2016/blob/master/ThaiPH/VirtualSwitch/Linux-bridge/ThaiPH_tim_hieu_linux_bridge.md

[3] - https://wiki.debian.org/BridgeNetworkConnections#Bridging_Network_Connections 