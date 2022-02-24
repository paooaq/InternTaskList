# Tìm hiểu về OpenvSwitch, so sánh với linux bridge

1. ## OpenvSwitch là gì

   ![../../_images/overview.png](https://docs.openvswitch.org/en/latest/_images/overview.png)

   - Open vSwitch là phần mềm switch mã nguồn mở hỗ trợ giao thức OpenFlow
   - Open vSwitch được sử dụng với các hypervisors để kết nối giữa các máy ảo trên một host vật lý và các máy ảo giữa các host vật lý khác nhau qua mạng.
   - Open vSwitch là một trong những thành phần quan trọng hỗ trợ SDN (Software Defined Networking - Công nghệ mạng điều khiển bằng phần mềm)
   - Tính năng:
     - Standard 802.1Q VLAN model with trunk and access ports
     - NIC bonding with or without LACP on upstream switch
     - NetFlow, sFlow(R), and mirroring for increased visibility
     - QoS (Quality of Service) configuration, plus policing
     - Geneve, GRE, VXLAN, STT, and LISP tunneling
     - 802.1ag connectivity fault management
     - OpenFlow 1.0 plus numerous extensions
     - Transactional configuration database with C and Python bindings
     - High-performance forwarding using a Linux kernel module

2. ## Các thành phần và kiến trúc của OpenvSwitch

   ### Các thành phần chính của OpenVSwitch:

   - ovs-vswitchd : daemon tạo ra switch, nó được đi kèm với Linux kernel module
   - ovsdb-server : Một máy chủ cơ sở dữ liệu nơi ovs-vswitchd truy vấn để có được cấu hình.
   - ovs-dpctl : công cụ để cấu hình switch kernel module.
   - ovs-vsctl : Dùng để truy vấn và cập nhật cấu hình cho ovs-vswitchd.
   - ovs-appctl : Dùng để gửi câu lệnh chạy Open vSwitch daemons.

![img](https://camo.githubusercontent.com/f52a116828821afcff6f31dc4b47fb35f83b9a7845eeeb2b27f86e6db759d890/687474703a2f2f692e696d6775722e636f6d2f427665695245592e6a7067)

### Một số công cụ do OpenvSwitch cung cấp:

- ovs-ofctl, một tiện ích để truy vấn và điều khiển các controller và OpenFlow switch.
- ovs-pki, một tiện ích để tạo và quản lý cơ sở hạ tầng khóa công khai cho các OpenFlow switch.
- ovs-testcontroller, một OpenFlow controller đơn giản có thể hữu ích cho việc thử nghiệm (mặc dù không dùng cho sản xuất).
- Một bản vá cho tcpdump cho phép nó phân tích cú pháp các tin nhắn OpenFlow.

### Cơ chế hoạt động

OVS kernel module sẽ dùng netlink socket để tương tác với vswitchd daemon để tạo và quản lí số lượng OVS switches trên hệ thống local. SDN Controller sẽ tương tác với vswitchd sử dụng giao thức OpenFlow. ovsdb-server chứa bảng dữ liệu. Các clients từ bên ngoài cũng có thể tương tác với ovsdb-server sử dụng json rpc với dữ liệu theo dạng file JSON.

Open vSwitch có 2 modes, normal và flow:

- Normal Mode: Ở mode này, Open vSwitch tự quản lí tất cả các công việc switching/forwarding. Nó hoạt động như một switch layer 2.
- Flow Mode: Ở mode này, Open vSwitch dùng flow table để quyết định xem port nào sẽ nhận packets. Flow table được quản lí bởi SDN controller nằm bên ngoài.

3. ## So sánh openvSwitch và Linux bridge

   | Open vSwitch                             | Linux bridge                                           |
   | ---------------------------------------- | ------------------------------------------------------ |
   | Được thiết kế cho môi trường mạng ảo hóa | Mục đích ban đầu không phải dành cho môi trường ảo hóa |
   | Có các chức năng của layer 2-4           | Chỉ có chức năng của layer 2                           |
   | Có khả năng mở rộng                      | Bị hạn chế về quy mô                                   |
   | ACLs, QoS, Bonding                       | Chỉ có chức năng forwarding                            |
   | Có OpenFlow Controller                   | Không phù hợp với môi trường cloud                     |
   | Hỗ trợ netflow và sflow                  | Không hỗ trợ tunneling                                 |

4. ## Ưu nhược điểm của OpenvSwitch và Linux Bridge

   #### OVS

   - Ưu điểm: các tính năng tích hợp nhiều và đa dạng, kế thừa từ linux bridge. OVS hỗ trợ ảo hóa lên tới layer4. Được sự hỗ trợ mạnh mẽ từ cộng đồng. Hỗ trợ xây dựng overlay network.
   - Nhược điểm: Phức tạp, gây ra xung đột luồng dữ liệu

   #### Linux Bridge

   - Ưu điểm:

   các tính năng chính của switch layer được tích hợp sẵn trong nhân. Có được sự ổn định và tin cậy, dễ dàng trong việc troubleshoot Less moving parts: được hiểu như LB hoạt động 1 cách đơn giản, các gói tin được forward nhanh chóng

   - Nhược điểm:

   để sử dụng ở mức user space phải cài đặt thêm các gói. VD vlan, ifenslave. Không hỗ trợ openflow và các giao thức điều khiển khác. không có được sự linh hoạt

