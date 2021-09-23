# Tìm hiểu về vxlan và gre protocol

1. ## Tìm hiểu về vxlan

   1. VXLAN là gì?

      - Virtual Extensible LAN (VXLAN) là giao thức tunneling, thuộc giữa lớp 2 và lớp 3.
      - VXLAN là giao thức sử dụng UDP (cổng 4789) để truyền thông và một segment ID độ dài 24 bit còn gọi là VXLAN network identifier (VNID). Chỉ các máy ảo trong cùng VXLAN segment mới có thể giao tiếp với nhau
      - VXLAN ID (VXLAN Network Identifier hoặc VNI) là 1 chuỗi 24-bits so với 12 bits của của VLAN ID. Do đó cung cấp hơn 16 triệu ID duy nhất ( giá trị này của VLAN: 4096 )

   2. VXLAN có những ưu điểm gì

      - VLAN sử dụng 12 bit VLAN ID tương đương với 4096 VLAN. VXLAN sử dụng 24-bit segment ID (VNID) tương đương với 16 triệu VLAN segment
      - VXLAN có thể mở rộng L2 segment trên 1 hạ tầng mạng dùng chung. Vì thế một thuê bao có thể chia workload ra nhiều server
      - Các gói VXLAN được vận chuyển trên hạ tầng mạng bằng L3 header nên có thể tận dụng L3

   3. Cái khái niệm trong  vxlan

      1. VNI

         - VXLAN hoạt động trên cơ sở hạ tầng mạng hiện có và cung cấp một phương tiện để "kéo dài" một mạng lớp 2. Tóm lại, VXLAN là một mạng lớp 2 overlay trên mạng lớp 3. Mỗi lớp mạng như vậy được gọi là VXLAN segment. Chỉ các máy ảo trong cùng VXLAN segment mới có thể giao tiếp với nhau. Mỗi VXLAN segment được xác định thông qua ID kích thước 24 bit, gọi là VXLAN Network Identifier (VNI). Điều này cho phép tối đa 16 triệu các VXLAN segment cùng tồn tại trong cùng một domain
         - VNI xác định phạm vi của inner MAC frame sinh ra bởi máy ảo VM. Do đó, bạn có thể overlapping địa chỉ MAC thông qua segment như không bị lẫn lộn các lưu lượng bởi chúng đã bị cô lập bởi VNI khác nhau. VNI nằm trong header được đóng gói với innere MAC sinh ra bởi VM.

      2. Encapsulation và VTEP

         VXLAN là công nghệ overlay qua lớp mạng. Overlay Network có thể được định nghĩa như là một mạng logic mà được tạo trên một nền tảng mạng vật lý đã có sẵn. VXLAN tạo một mạng vật lý layer 2 trên lớp mạng IP. Dưới đây là 2 từ khóa được dùng trong công nghệ overlay network:

         - Encapsulate: Đóng gói những gói tin ethernet thông thường trong một header mới. Ví dụ: trong công nghệ overlay IPSec VPN, đóng gói gói tin IP thông thường vào một IP header khác
         - VTEP: Việc liên lạc được thiết lập giữa 2 đầu tunnel end points (đường ống).

         Khi áp dụng vào với công nghệ overlay trong VXLAN, VXLAN sẽ đóng gói một frame MAC thông thường vào một UDP header. Và tất cả các host tham gia vào VXLAN thì hoạt động như một tunnel end points. Chúng gọi là Virtual Tunnel Endpoints (VTEPs).

         ![vm-to-vm.png](https://github.com/thangtq710/GRE-VXLAN-protocol/blob/master/images/vm-to-vm.png?raw=true)

         VXLAN học tất cả các địa chỉ MAC của máy ảo và việc kết nối nó tới VTEP IP thì được thực hiện thông qua sự hỗ trợ của mạng vật lý. Một trong những giao thức được sử dụng trong mạng vật lý là IP multicast. VXLAN sử dụng giao thức của IP multicast để cư trú trong bảng forwarding trong VTEP.

         Do sự đóng gói (encapsulation) này, VXLAN có thể được gọi là thiết lập đường hầm (tunneling) để kéo dài mạng lớp 2 thông qua lớp 3. Điểm cuối các tunnel này - (VXLAN Tunnel End Point hoặc VTEP) nằm trong hypervisor trên server máy chủ của các VM. Do đó, VNI và VXLAN liên quan tới các khái niệm đóng gói header tunnel được thực hiện bởi VTEP - và trong suốt với VM.

      3. VXLAN frame format

         ![vxlanframe.png](https://github.com/thangtq710/GRE-VXLAN-protocol/blob/master/images/vxlanframe.png?raw=true)

         Frame Ethernet thông thường bao gồm địa chỉ MAC nguồn, MAC đích, Ethernet type và thêm phần VLAN_ID (802.1q) nếu có. Đây là frame được đóng gói sử dụng VXLAN, thêm các header sau:

         - VXLAN header: 8 byte bao gồm các trường quan trọng sau:
           - Flags: 8 but, trong đó bit thứ 5 (I flag) được thiết lập là 1 để chỉ ra rằng đó là một frame có VNI có giá trị. 7 bit còn lại dùng dữ trữ được thiết lập là 0 hết.
           - VNI: 24 bit cung cấp định danh duy nhất cho VXLAN segment. Các VM trong các VXLAN khác nhau không thể giao tiếp với nhau. 24 bit VNI cung cấp lên tới hơn 16 triệu VXLAN segment trong một vùng quản trị mạng.
         - Outer UDP Header: port nguồn của Outer UDP được gán tự động và sinh ra bởi VTEP và port đích thông thường được sử dụng là port 4789 hay được sử dụng (có thể chọn port khác).
         - Outer IP Header: Cung cấp địa chỉ IP nguồn của VTEP nguồn kết nối với VM bên trong. Địa chỉ IP outer đích là địa chỉ IP của VTEP nhận frame.
         - Outer Ethernet Header: cung cấp địa chỉ MAC nguồn của VTEP có khung frame ban đầu. Địa chỉ MAC đích là địa chỉ của hop tiếp theo được định tuyến bởi VTEP.

2. ## Tìm hiểu về gre protocol

   1. ### GRE là gì?

      - **Generic routing encapsulation (GRE)** là một giao thức sử dụng để thiết lập các kết nối ***point-to-point*** một các trực tiếp giữa các node trong mạng. Đây là một phương pháp đơn giản và hiệu quả để chuyển dữ liệu thông qua mạng public network, như Internet. GRE cho phép hai bên chia sẻ dữ liệu mà họ không thể chia sẻ với nhau thông qua mạng public network.
      - GRE đóng gói dữ liệu và chuyển trực tiếp tới thiết bị mà de-encapsulate gói tin và định tuyến chúng tới đích cuối cùng. Gói tin và định tuyến chúng tới đích cuối cùng. GRE cho phép các switch nguồn và đích hoạt động như một kết nối ảo point-to-point với các thiết bị khác (bởi vì outer header được áp dụng với GRE thì trong suốt với payload được đóng gói bên trong).

      Ví dụ: GRE tạo tunnel cho phép các giao thức định tuyến như RIP và OSPF chuyển tiếp các gói tin từ một switch tới switch khác thông qua mạng Internet. Hơn nữa, GRE tunnel có thể đóng gói các dữ liệu truyền multicast để truyền thông qua Internet.

   2. ### Các ưu điểm của GRE

      - Cho phép đóng gói nhiều giao thức và truyền thông qua một giao thức backbone (IP protocol).
      - Cung cấp cách giải quyết cho các mạng bị hạn chế hop (hạn chế số hop di chuyển tối đa trong một mạng).
      - Kết nối các mạng con gián tiếp.
      - Yêu cầu ít tài nguyên hơn các giải pháp tunnel khác. (ví dụ Ipsec VPN).

   3. ### GRE tunneling

      - Dữ liệu được định tuyến bởi hệ thống GRE endpoint trên các tuyến đường được thiết lập trong bảng tuyến. (Các tuyến này có thể được cấu hình tĩnh hoặc học động bằng các giao thức định tuyến như RIP hoặc OSPF) Khi một gói dữ liệu nhận được bởi GRE endpoint, nó được decapsulation và định tuyến lại đến địa chỉ đích cuối cùng của nó.
      - GRE tunnel là ***stateless*** - nghĩa là tunnel endpoint không chứa thông tin về trạng thái hoặc tính sẵn có của remote tunnel endpoint. Do đó, switch hoạt động như một tunnel source router không thể thay đổi trạng thái của GRE tunnel interface thành down nếu remote endpoint không thể truy cập được. (Hiểu như là không hoạt động kiểu cần phải thiết lập kết nối trước khi truyền dữ liệu như TCP)

   4. ### Encapsulation và De-Encapsulation trên switch

      - **Encapsulation**

        Switch hoạt động như một tunnel source router đóng gói và chuyển tiếp các gói tin GRE như sau:

        - Switch nhận dữ liệu gói tin (payload) cần chuyển qua tunnel, nó sẽ chuyển gói tin ra tunnel interface.
        - Tunnel interface đóng gói
        - Encapsulate dữ liệu vào trong gói tin GRE và thêm vào đó phần outer IP header để thành gói tin IP mới.
        - Gói tin IP được chuyển đến địa chỉ IP đích trong phần outer IP header (là địa chỉ IP của tunnel interface nhận)

      - **De-encapsulation**

        **De-encapsulation** switch hoạt động như một tunnel remote router xử lý gói tin GRE như sau:

        - Khi đích outer IP nhận được gói tin từ tunnel interface, outer IP header và GRE header sẽ được bóc tách khỏi gói tin.
        - Gói tin được định tuyến tới địa chỉ đích cuối cùng dựa vào inner IP header.

   5. #### GRE frame format

      GRE thêm vào tối thiểu 24 byte vào gói tin, trong đó bao gồm 20-byte IP header mới, 4 byte còn lại là GRE header. GRE có thể tùy chọn thêm vào 12 byte mở rộng để cung cấp tính năng tin cậy như: checksum, key chứng thực, sequence number.

      ![6.1.png](https://github.com/hocchudong/thuctap012017/blob/master/TamNT/Virtualization/images/6.1.png?raw=true)

      GRE header bản thân nó chứa đựng 4 byte, đây là kích cỡ nhỏ nhất của một GRE header khi không thêm vào các tùy chọn. 2 byte đầu tiên là các cờ (flags) để chỉ định những tùy chọn GRE. Những tùy chọn này nếu được active, nó thêm vào GRE header. Bảng sau mô tả những tùy chọn của GRE header.

      ![6.2.png](https://github.com/hocchudong/thuctap012017/blob/master/TamNT/Virtualization/images/6.2.png?raw=true)

      Trong GRE header 2 byte còn lại chỉ định cho trường giao thức. 16 bits này xác định kiểu của gói tin được mang theo trong GRE tunnel. Hình sau mô tả cách mà một gói tin GRE với tất cả tùy chọn được gán vào một IP header và data:

      ![6.3.png](https://github.com/hocchudong/thuctap012017/blob/master/TamNT/Virtualization/images/6.3.png?raw=true)

      Khi được encapsulation qua GRE tunnel, kích thước bản tin tăng thêm 4 + 20 + 14 = 38 bytes

   6. #### Phân loại GRE

      - #### Point-to-Point GRE

        Đối với các tunnel GRE point-to-point thì trên mỗi router spoke (R2 & R3) cấu hình một tunnel chỉ đến HUB (R1) ngược lại, trên router HUB cũng sẽ phải cấu hình hai tunnel, một đến R2 và một đến R3. Mỗi tunnel như vậy thì cần một địa chỉ IP. Giả sử mô hình trên được mở rộng thành nhiều spoke, thì trên R1 cần phải cấu hình phức tạp và tốn không gian địa chỉ IP.

        ![6.4.png](https://github.com/hocchudong/thuctap012017/blob/master/TamNT/Virtualization/images/6.4.png?raw=true)

        Trong tunnel GRE point-To-point, điểm đầu và cuối được xác định thì có thể truyền dữ liệu. Tuy nhiên, có một vấn đề phát sinh là nếu địa chỉ đích là một multicast (chẳng hạn 224.0.0.5) thì GRE point-to-point không thực hiện được. Để làm được việc này thì phải cần đến mGRE.

      - #### Point-to-Multipoint GRE (mGRE)

        mGRE giải quyết được vấn đề đích đến là một địa chỉ multicast. Đây là tính năng chính của mGRE được dùng để thực thi Multicast VPN trong Cisco IOS. Tuy nhiên, trong mGRE, điểm cuối chưa được xác định nên nó cần một giao thức để ánh xạ địa chỉ tunnel sang địa chỉ cổng vật lý. Giao thức này được gọi là NHRP (Next Hop Resolution Protocol)

        ![6.5.png](https://github.com/hocchudong/thuctap012017/blob/master/TamNT/Virtualization/images/6.5.png?raw=true)

        mGRE Tunnel thì mỗi router chỉ có một Tunnel được cấu hình cùng một subnet logical.