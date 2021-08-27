## Tìm hiểu KVM (Kernel-based Virtual Machine)

1. ### Khái niệm

   KVM (Kernel-based virtual machine) là giải pháp ảo hóa cho hệ thống linux trên nền tảng phần cứng x86 có các module mở rộng hỗ trợ ảo hóa (Intel VT-x hoặc AMD-V). 

   Về bản chất, KVM không thực sự là một hypervisor có chức năng giải lập phần cứng để chạy các máy ảo. Chính xác KVM chỉ là một module của kernel linux hỗ trợ cơ chế mapping các chỉ dẫn trên CPU ảo (của guest VM) sang chỉ dẫn trên CPU vật lý (của máy chủ chứa VM). Hoặc có thể hình dung KVM giống như một driver cho hypervisor để sử dụng được tính năng ảo hóa của các vi xử lý như Intel VT-x hay AMD-V, mục tiêu là tăng hiệu suất cho guest VM.

2. ### KVM Stack

   ![image-20210827114938289](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20210827114938289.png)

   KVM Stack bao gồm 4 tầng:

   - User-facing tools: Là các công cụ quản lý máy ảo hỗ trợ KVM. Các công cụ có giao diện đồ họa (như virt-manager) hoặc giao diện dòng lệnh như (virsh)
   - Management layer: Lớp này là thư viện libvirt cung cấp API để các công cụ quản lý máy ảo hoặc các hypervisor tương tác với KVM thực hiện các thao tác quản lý tài nguyên ảo hóa, vì tự thân KVM không hề có khả năng giả lập và quản lý tài nguyên như vậy.
   - Virtual machine: Chính là các máy ảo người dùng tạo ra. Thông thường, nếu không sử dụng các công cụ như virsh hay virt-manager, KVM sẽ sử được sử dụng phối hợp với một hypervisor khác điển hình là QEMU.
   - Kernel support: Chính là KVM, cung cấp một module làm hạt nhân cho hạ tầng ảo hóa (kvm.ko) và một module kernel đặc biệt hỗ trợ các vi xử lý VT-x hoặc AMD-V (kvm-intel.ko hoặc kvm-amd.ko)

3. ## QEMU

   1. ### QEMU as an emulator

      Khi QEMU hoạt động như một trình giả lập, nó có khả năng chạy các hệ điều hành / chương trình được tạo cho một loại máy trên một loại máy khác nhau.Nó chỉ sử dụng các phương pháp dịch nhị phân. Trong chế độ này, QEMU mô phỏng CPU thông qua các kỹ thuật dịch nhị phân động và cung cấp một tập hợp các mô hình thiết bị. Do đó, nó được kích hoạt để chạy các hệ điều hành khách không sửa đổi khác nhau với các kiến trúc khác nhau. Bản dịch nhị phân là cần thiết ở đây vì mã khách có được thực thi trong CPU chủ. Trình dịch nhị phân thực hiện công việc này được gọi là Tiny 
      Code Generator (TCG); nó là một trình biên dịch Just-In-Time (JIT). Nó biến đổi mã nhị phân được viết cho một bộ xử lý nhất định thành một dạng mã nhị phân khác:

      ![image-20210827124157609](https://github.com/paooaq/InternTaskList/image/qemu_as_emulator)

      Bằng cách sử dụng phương pháp này, QEMU có thể hy sinh một chút tốc độ thực thi cho phạm vi rộng hơn nhiều
      tính tương thích.

   2. ### QEMU as a virtualizer

      QEMU là có khả năng chạy mà không cần KVM bằng cách sử dụng phương pháp dịch nhị phân nói trên.
      Quá trình thực thi này sẽ chậm hơn so với ảo hóa tăng tốc phần cứng được bật bởi KVM. Trong bất kỳ chế độ nào, với tư cách là trình ảo hóa hoặc trình giả lập, QEMU không chỉ mô phỏng bộ xử lý; nó cũng mô phỏng các thiết bị ngoại vi khác nhau, chẳng hạn như đĩa, mạng, VGA, PCI, cổng nối tiếp và song song, USB, v.v. Ngoài mô phỏng thiết bị I/O này, khi làm việc với KVM, QEMU-KVM tạo và khởi tạo các máy ảo. Như hình trong sơ đồ sau, nó cũng khởi tạo các luồng POSIX khác nhau cho mỗi CPU (vCPU) của khách. Nó cũng cung cấp một khuôn khổ được sử dụng để mô phỏng không gian địa chỉ vật lý của máy trong không gian địa chỉ chế độ người dùng của QEMU-KVM:

      ![](https://github.com/paooaq/InternTaskList/image/QEMU_as_a_virtualizer)

4. ## KVM - QEMU

   \- Hệ thống ảo hóa KVM hay đi liền với QEMU. Về mặt bản chất, QEMU là một emulator. QEMU có khả năng giả lập tài nguyên phần cứng, trong đó bao gồm một CPU ảo. Các chỉ dẫn của hệ điều hành tác động lên CPU ảo này sẽ được QEMU chuyển đổi thành chỉ dẫn lên CPU vật lý nhờ một translator là TCG(Tiny Core Generator) nhưng TCG hiệu suất ko cao.
   \- Do KVM hỗ trợ ánh xạ CPU vật lý sang CPU ảo, cung cấp khả năng tăng tốc phần cứng cho máy ảo và hiệu suất của nó nên QEMU sử dụng KVM làm accelerator tận dụng tính năng này của KVM thay vì sử dụng TCG.

   1. ### Các tool để điều khiển KVM-QEMU

      Đối với từng dạng ảo hóa như Kvm, Xen, .. sẽ có một tiến trình Libvirt chạy để điều khiển các dang ảo hóa và cung cấp những API để các tool như virsh, virt-manager, Openstack, ovirt có thể giao tiếp với KVM-Qemu thông qua livbirt.

      ![img](https://camo.githubusercontent.com/f213e7d48c1641ad15f9988ed731c354abb92a8e01997224b8d80405d2141c10/687474703a2f2f692e696d6775722e636f6d2f6332516e3456382e706e67)

   2. ### Tính năng Migrate trong KVM-QEMU

      Migrate là chức năng được KVM-QEMU hỗ trợ, nó cho phép di chuyển các guest từ một host vật lý này sang host vật lý khác và không ảnh hướng để guest đang chạy cũng như dữ liệu bên trong nó

      Migrate giúp cho nhà quản trị có thể di chuyển các guest trên host đi để phục vụ cho việc bảo trì và nâng cấp hệ thống, nó cũng giúp nhà quản trị nâng cao tính dự phòng, và cũng có thể làm nhiệm vụ load bandsing cho hệ thống khi một máy host quá tải

      Migrate có 2 cơ chế:

      - Cơ chế Offline Migrate: là cơ chế cần phải tắt guest đi thực hiện việc di chuyển image và file xml của guest sang một host khác Mô hình thuần túy của cơ chế Offline Migrate
      - Cơ chế Live Migrate: đây là cơ chế di chuyển guest khi guest vẫn đang hoạt động, quá trình trao đổi diễn ra rất nhanh các phiên làm việc kết nối hầu như không cảm nhận được sự gián đoạn nào. Quá trình Live Migrate được diễn ra như sau: Bước đầu tiên của quá trình Live Migrate 1 ảnh chụp ban đầu của guest trên host1 được chuyển sang host2. Trong trường hợp người dùng đang truy cập tại host1 thì những sự thay đổi và hoạt động trên host1 vẫn diễn ra bình thường, tuy nhiên những thay đổi này sẽ được ghi nhận. Những thay đổi trên host1 được đồng bộ liên tục đến host2 Khi đã đồng bộ xong thì guest trên host1 sẽ offline và các phiên truy cập trên host1 được chuyển sang host2.

5. ## Các tính năng của KVM

   1. ### Security

      ![img](https://camo.githubusercontent.com/b9c22845e52c52d6b2a5ed690825b39c3dc264c35f1d995619949ae35d1fe4e4/687474703a2f2f692e696d6775722e636f6d2f39314756674d4d2e706e67)

      \- Trong kiến trúc KVM, máy ảo được xem như các tiến trình Linux thông thường, nhờ đó nó tận dụng được mô hình bảo mật của hệ thống Linux như SELinux, cung cấp khả năng cô lập và kiểm soát tài nguyên.
      \- Bên cạnh đó còn có SVirt project - dự án cung cấp giải pháp bảo mật MAC (Mandatory Access Control - Kiểm soát truy cập bắt buộc) tích hợp với hệ thống ảo hóa sử dụng SELinux để cung cấp một cơ sở hạ tầng cho phép người quản trị định nghĩa nên các chính sách để cô lập các máy ảo. Nghĩa là SVirt sẽ đảm bảo rằng các tài nguyên của máy ảo không thể bị truy cập bởi bất kì các tiến trình nào khác; việc này cũng có thể thay đổi bởi người quản trị hệ thống để đặt ra quyền hạn đặc biệt, nhóm các máy ảo với nhau chia sẻ chung tài nguyên.

   2. ### Memory Management

      \- KVM thừa kế tính năng quản lý bộ nhớ mạnh mẽ của Linux. Vùng nhớ của máy ảo được lưu trữ trên cùng một vùng nhớ dành cho các tiến trình Linux khác và có thể swap. KVM hỗ trợ NUMA (Non-Uniform Memory Access - bộ nhớ thiết kế cho hệ thống đa xử lý) cho phép tận dụng hiệu quả vùng nhớ kích thước lớn.
      \- KVM hỗ trợ các tính năng ảo của mới nhất từ các nhà cung cấp CPU như EPT (Extended Page Table) của Microsoft, Rapid Virtualization Indexing (RVI) của AMD để giảm thiểu mức độ sử dụng CPU và cho thông lượng cao hơn.
      \- KVM cũng hỗ trợ tính năng Memory page sharing bằng cách sử dụng tính năng của kernel là Kernel Same-page Merging (KSM).

   3. ### Storage

      \- KVM có khả năng sử dụng bất kỳ giải pháp lưu trữ nào hỗ trợ bởi Linux để lưu trữ các Images của các máy ảo, bao gồm các ổ cục bộ như IDE, SCSI và SATA, Network Attached Storage (NAS) bao gồm NFS và SAMBA/CIFS, hoặc SAN thông qua các giao thức iSCSI và Fibre Channel.
      \- KVM tận dụng được các hệ thống lưu trữ tin cậy từ các nhà cung cấp hàng đầu trong lĩnh vực Storage.
      \- KVM cũng hỗ trợ các images của các máy ảo trên hệ thống tệp tin chia sẻ như GFS2 cho phép các images có thể được chia sẻ giữa nhiều host hoặc chia sẻ chung giữa các ổ logic.

   4. ### Live migration

      \- KVM hỗ trợ live migration cung cấp khả năng di chuyển ác máy ảo đang chạy giữa các host vật lý mà không làm gián đoạn dịch vụ. Khả năng live migration là trong suốt với người dùng, các máy ảo vẫn duy trì trạng thái bật, kết nối mạng vẫn đảm bảo và các ứng dụng của người dùng vẫn tiếp tục duy trì trong khi máy ảo được đưa sang một host vật lý mới. KVM cũng cho phép lưu lại trạng thái hiện tại của máy ảo để cho phép lưu trữ và khôi phục trạng thái đó vào lần sử dụng tiếp theo.

   5. ### Performance and scalability

      \- KVM kế thừa hiệu năng và khả năng mở rộng của Linux, hỗ trợ máy ảo với 16 CPUs ảo, 256GB RAM và hệ thống máy host lên tới 256 cores và trên 1TB RAM.

