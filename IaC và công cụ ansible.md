# IaC (Infrastructure as code)

#### Khái niệm: Infrastructure as code - Cơ sở hạ tầng như mã code, còn được gọi là IaC, là một mô hình thiết lập và triển khai công nghệ trong đó các nhà phát triển hoặc nhóm vận hành tự động quản lý và cung cấp các gói công nghệ cho ứng dụng thông qua phần mềm, thay vì sử dụng quy trình thủ công để cấu hình các thiết bị phần cứng và hệ điều hành rời rạc như trước kia.

#### Lợi ích của Infrastructure-as-code: 

- [ ] Có thể viết một quy trình Infrastructure-as-code để xuất bản và triển khai ứng dụng mới để đảm bảo chất lượng hoặc triển khai thử nghiệm trước khi chuyển giao cho bên tiếp quản triển khai trực tiếp trong sản xuất
- [ ]  Thiết lập cơ sở hạ tầng được viết dưới dạng mã, có thể trải qua cùng một phiên bản quản trị, kiểm tra tự động và các bước khác trong pipeline tích hợp liên tục và phân phối liên tục (CI/CD) mà các nhà phát triển vẫn sử dụng cho mã ứng dụng.
- [ ] Có thể chọn kết hợp cơ sở hạ tầng dưới dạng mã với các container, để tách ứng dụng khỏi cơ sở hạ tầng ở cấp hệ điều hành.

#### Bất lợi của Infrastructure-as-code:

- [ ] Công nghệ đòi hỏi các công cụ bổ sung, chẳng hạn như một hệ thống quản lý cấu hình, có thể đưa vào các learning curve và không gian lỗi.
- [ ] Bất kỳ lỗi nào cũng có thể sinh sôi và lan truyền nhanh chóng qua hệ thống các máy chủ, do đó, cần thiết phải giám sát và kiểm soát toàn bộ phiên bản cũng như thực hiện kiểm tra trước khi phát hành toàn diện.
- [ ] Quản trị viên thay đổi cấu hình máy chủ không theo cấu hình Infrastructure-as-code đã định trước, có thể xảy ra tình trạng bị trôi cấu hình.

# Công cụ ansible

Việc cài đặt và cấu hình các máy chủ thường được ghi chép lại trong tài liệu dưới dạng các câu lệnh đã chạy, với giải thích kèm theo. Cách thức này gây mệt mỏi cho quản trị viên vì phải làm theo từng bước ở mỗi máy khi thiết lập mới, và có thể dẫn đến sai lầm, thiếu sót. 

####  => Ansible giúp cấu hình server theo tùy biến rất đa dạng, giảm thiểu thời gian thao tác trên từng server được cài đặt

1. ##### Khái niệm ansible

   - Là công cụ mã nguồn mở dùng để quản lý cài đặt, cấu hình hệ thống một cách tập trung và cho phép thực thi câu lệnh điều khiển.
   - Sử dụng SSH (hoặc Powershell) và các module được viết bằng ngôn ngữ Python để điểu khiển hệ thống.
   - Sử dụng định dạng JSON để hiển thị thông tin và sử dụng YAML (Yet Another Markup Language) để xây dựng cấu trúc mô tả hệ thống.
   
2. ##### Cách cài đặt ansible

   - **Cài đặt trên ubuntu**

     apt-add-repository -y ppa:ansible/ansible 

     apt-get update 

     sudo apt-get install ansible -y

   - **Cài đặt trên centOS**

     yum install epel-release

     yum install git python python-devel python-pip openssl ansible

3. ##### Cấu hình ansible

   - ###### Tạo tài khoản truy cập SSH trên agent

     Do policy của hệ thống giới hạn tài khoản root truy cập cũng như để thuận tiện cho việc quản lý (tránh dùng chung account của người quản trị) thì nên tạo 1 tài khoản khác phục vụ cho ansible

     1. Tạo tài khoản : **sudo adduser ansible**

     2. Cấu hình cho user ansible sử dụng không cần password:

        **$ sudo vi /etc/sudoers.d/ansible**
        ansible ALL=(ALL)   NOPASSWD:ALL

   - ###### Tạo ssh key

     1. Tạo ssh keyfile: ( trên master )

        **$ ssh-keygen -C "ansible@master"**
        Enter file in which to save the key (/home/ubuntu/.ssh/id_rsa): `/etc/ansible/ansible_key`  
        ...

        ***=>không đặt ssh key ở thư mục mặc định do rất có thể có nhiều người quản trị cùng tham gia quản lý***

     2. Copy keyfile sang agent:
	vi /etc/ssh/sshd_config => PasswordAuthentication yes
	service sshd reload

        **$ ssh-copy-id -i /etc/ansible/ansible_key.pub ansible@[ip_server_agent]**

     3. Kiểm tra :

        **$ ssh -i /etc/ansible/ansible_key ansible@[ip_server_agent]** 

   - ###### **Cấu hình host và group**

     Ansible lưu thông tin những hệ thống trong file /etc/ansible/hosts (inventory) theo cấu trúc dạng INI như sau:

     > **mail.example.com**
     >  **[webservers]** 
     > **foo.example.com 
     > bar.example.com  [dbservers]** 
     > **two.example.com** 
     > **one.example.com** 
     > **three.example.com**

     Trong đó: .example.com: hostname của các host (phải cấu hình tiếp trong file /etc/hosts để xác định địa chỉ IP)[webservers], [dbservers]: là các group

4. ##### Cấu trúc lệnh gọi ansible

   - **ansible [tên host] -m [tên module] -a [tham số truyền vào module]**
   - Một số câu lệnh cơ bản:
     1. ansible all -m ping (giải thích: gọi ping toàn bộ các hosts trong /etc/ansible/hosts)
     2. ansible all -m command -a uptime(Default, ansible sẽ cho module = "command". Nên ta ko cần -m command thêm vào cũng được.)

   - Tất cả module của ansible bạn có thể tham khảo ở đây [Tài liệu](https://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html#default-groups)

5. **Ansible playbook**

   Ansible rất linh hoạt khi hỗ trợ playbook bằng ngôn ngữ YAML (file .yml). Từ đó, khi admin cần setup server/service nào. Chỉ cần gọi file yml này ra, tất cả sẽ được thực thi một cách tự động.

   Mỗi một thao tác trong Playbook gọi là 1 task, ta sử dụng module để tạo thành task

   
