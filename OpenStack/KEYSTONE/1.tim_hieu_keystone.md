# Tìm hiểu về keystone



1. ## Keystone là gì

   Keystone là một Openstack project cung cấp các dịch vụ Identify, Token, Catalog, Policy cho các dịch vụ khác trong Openstack. Keystone gồm hai phiên bản: 

   - **v2**: sử dụng UUID
   - **v3**: sử dụng PKI, sử dụng một cặp key mở và đóng để xác minh chéo và xác thực. Hai tính năng chính của Keystone

   Hai tính năng chính của Keystone:

   - User Management: Keystone giúp xác thực tài khoản người dùng và chỉ định xem người dùng có quyền gì.
   - Service Catalog: Cung cấp một danh mục các dịch vụ sẵn sàng cùng với các API endpoint để truy cập các dịch vụ đó.

2. ## Các khái niệm liên quan

   - **Authentication**: Là quá trình xác nhận danh tính của người dùng dựa trên thông tin đăng nhập của người đó(**credential**). Khi xác thực được danh tính người dùng, nó cấp cho người dùng một token xác thực. Người dùng sẽ cung cấp token đó cho mỗi yêu cầu sau đó.
   - **Credentials**: Là thông tin dùng để xác thực người dùng. Ví dụ như username, password và API key, hay là token mà được cung cấp.
   - **Domain**: Là một thực thể API v3 dịch vụ nhận dạng, tập hợp của các project của người dùng để xác định quản trị xác thực. Có thể là một cá nhân hay tổ chức, hoặc của nhà quản trị.
   - **Endpoint**: Là một địa chỉ truy cập mạng, thường là địa chỉ URL của một dịch vụ để từ đó truy cập vào dịch vụ.
   - **Group**: Là một thực thể API v3 dịch vụ nhận dạng, là một nhóm những người dùng nằm trong một domain. Quyền của một group được thêm vào một domain hay một project sẽ được áp dụng cho tất cả user của group đó.
   - **Openstackclient**: Là một công cụ dòng lệnh cung cấp giao diện để truy cập các dịch vụ Openstack.
   - **Project**: sử dụng để nhóm hoặc cô lập tài nguyên, hoặc định danh các đối tượng. Tùy thuộc vào nhà quản lý.
   - **Region**: Là một thực thể API v3 dịch vụ nhận dạng, đại diện cho một bộ phận chung trong triển khai Openstack. Có thể tạo và liên kết chúng với các region phụ để tạo cấu trúc dạng cây. Region không có ý nghĩa về mặt địa lý nhưng tên của region thường được đặt theo tên khu vực địa lý(vd: asia-east)
   - **Roles**: Là tập hợp các quyền hạn và đặc quyền của người dùng để thực hiện các hành động cụ thể. Token được gửi đến người dùng sau khi xác thực sẽ bao gồm cả một tập hợp các quyền. Khi người dùng yêu cầu một dịch vụ, dịch vụ này sẽ kiểm tra quyền hạn của người dùng và cung cấp dịch vụ trong quyền hạn đó.
   - **Service**: Một dịch vụ Openstack, như Compute (nova), Object Storage (swift), hay Image service (glance), là một hay nhiều endpoint mà qua đó người dùng có thể truy cập tài nguyên và thực hiện các hành động.
   - **Token**: Một chuỗi gồm chữ và chữ số cho phép truy cập vào các tài nguyên và API Openstack. Token có thể bị thu hồi bất kỳ lúc nào và có giá trị trong thời gian hữu hạn.
   - **User**: Đại diện cho một người dùng, hệ thống hay dịch vụ mà sử dụng dịch vụ Openstack

3. ## Chức năng chính của keystone

   1. **Identity**
      - Nhận diện những người đang cố truy cập vào các tài nguyên cloud
      - Trong keystone, identity thường được hiểu là User
      - Tại những mô hình OpenStack nhỏ, identity của user thường được lưu trữ trong database của keystone. Đối với những mô hình lớn cho doanh nghiệp thì 1 external Identity Provider thường được sử dụng.
   2. **Authentication**
      - Là quá trình xác thực những thông tin dùng để nhận định user (user's identity)
      - keystone có tính pluggable tức là nó có thể liên kết với những dịch vụ xác thực người dùng khác như LDAP hoặc Active Directory.
      - Thường thì keystone sử dụng Password cho việc xác thực người dùng. Đối với những phần còn lại, keystone sử dụng tokens.
      - OpenStack dựa rất nhiều vào tokens để xác thực và keystone chính là dịch vụ duy nhất có thể tạo ra tokens
      - Token có giới hạn về thời gian được phép sử dụng. Khi token hết hạn thì user sẽ được cấp một token mới. Cơ chế này làm giảm nguy cơ user bị đánh cắp token.
      - Hiện tại, keystone đang sử dụng cơ chế bearer token. Có nghĩa là bất cứ ai có token thì sẽ có khả năng truy cập vào tài nguyên của cloud.
   3. **Access Management (Authorization)**
      - Access Management hay còn được gọi là Authorization là quá trình xác định những tài nguyên mà user được phép truy cập tới.
      - Trong OpenStack, keystone kết nối users với những Projects hoặc Domains bằng cách gán role cho user vào những project hoặc domain ấy.
      - Các projects trong OpenStack như Nova, Cinder...sẽ kiểm tra mối quan hệ giữa role và các user's project và xác định giá trị của những thông tin này theo cơ chế các quy định (policy engine). Policy engine sẽ tự động kiểm tra các thông tin (thường là role) và xác định xem user được phép thực hiện những gì.
   4. **Các lợi ích khi sử dụng keystone**
      - Cung cấp giao diện xác thực và quản lí truy cập cho các services của OpenStack. Nó cũng đồng thời lo toàn bộ việc giao tiếp và làm việc với các hệ thống xác thực bên ngoài.
      - Cung cấp danh sách đăng kí các containers (“Projects”) mà nhờ vậy các services khác của OpenStack có thể dùng nó để "tách" tài nguyên (servers, images,...)
      - Cung cấp danh sách đăng kí các Domains được dùng để định nghĩa các khu vực riêng biệt cho users, groups, và projects khiến các khách hàng trở nên "tách biệt" với nhau.
      - Danh sách đăng kí các Roles được keystone dùng để ủy quyền cho các services của OpenStack thông qua file policy.
      - Assignment store cho phép users và groups được gán các roles trong projects và domains.
      - Catalog lưu trưc các services của OpenStack, endpoints và regions, cho phép người dùng tìm kiếm các endpoints của services mà họ cần.

4. ## Identify service

   **Identify service**(dịch vụ xác thực) trong keystone được cung cấp bởi các **Actors**. Nó có thể tới từ nhiều dịch vụ như SQL, LDAP và Federated Identify Providers.

   1. ##### **SQL**

      - Keystone có tùy chọn cho phép lưu trữ actors trong SQL. Nó hỗ trợ các database như MySQL, PostgreSQL, và DB2.
      - Keystone sẽ lưu những thông tin như tên, mật khẩu và mô tả.
      - Những cài đặt của các database này nằm trong file config của keystone.
      - Về bản chất, Keystone sẽ hoạt động như 1 Identity Provider. Vì thế đây sẽ không phải là lựa chọn tốt nhất trong một vài trường hợp, nhất là đối với các khách hàng là doanh nghiệp.
      - Sau đây là ưu nhược điểm:

      Ưu điểm:

      - Dễ set up
      - Quản lí users, groups thông qua OpenStack APIs.

      Nhược điểm:

      - Keystone không nên là một Identity Provider
      - Hỗ trợ cả mật khẩu yếu
      - Hầu hết các doanh nghiệp đều sử dụng LDAP server
      - Phải ghi nhớ username và password.

   2. #####  LDAP

      - Keystone cũng có tùy chọn cho phép lưu trữ actors trong Lightweight Directory Access Protocol (LDAP).
      - Keystone sẽ truy cập tới LDAP như bất kì ứng dụng khác (System Login, Email, Web Application, etc.).
      - Các cài đặt kết nối sẽ được lưu trong file config của keystone. Các cài đặt này cũng bao gồm tùy chọn cho phép keystone được ghi hoặc chỉ đọc dữ liệu từ LDAP.
      - Thông thường LDAP chỉ nên cho phép các câu lệnh đọc, ví dụ như tìm kiếm user, group và xác thực.
      - Nếu sử dụng LDAP như một read-only Identity Backends thì Keystone cần có quyền sử dụng LDAP.
      - Một số ưu nhược điểm:

      Ưu điểm:

      - Không cần sao lưu tài khoản người dùng.
      - Keystone không hoạt động như một Identity Provider.

      Nhược điểm:

      - Account của các dịch vụ sẽ lưu ở đâu đó và người quản trị LDAP không muốn có tài khoản này trong LDAP.
      - Keystone có thể thấy mật khẩu người dùng, lúc mật khẩu được yêu cầu authentication.

   3. ##### Multiple backends

      - Kể từ bản Juno thì Keystone đã hỗ trợ nhiều Identity backends cho V3 Identity API. Nhờ vậy mà mỗi một domain có thể có một identity source (backend) khác nhau.
      - Domain mặc định thường sử dụng SQL backend bởi nó được dùng để lưu các host service accounts. Service accounts là các tài khoản được dùng bởi các dịch vụ OpenStack khác nhau để tương tác với Keystone.
      - Việc sử dụng Multiple Backends được lấy cảm hứng trong các môi trường doanh nghiệp, LDAP chỉ được sử dụng để lưu thông tin của các nhân viên bởi LDAP admin có thể không ở cùng một công ty với nhóm triển khai OpenStack. Bên cạnh đó, nhiều LDAP cũng có thể được sử dụng trong trường hợp công ty có nhiều phòng ban.
      - Sau đây là một số ưu nhược điểm:

      Ưu điểm:

      - Cho phép việc sử dụng nhiều LDAP để lưu tài khoản người dùng và SQL để lưu tài khoản dịch vụ
      - Sử dụng lại LDAP đã có.

      Nhược điểm:

      - Phức tạp trong khâu set up
      - Xác thực tài khoản người dùng phải trong miền scoped

   4. ##### Identity Providers

      - Kể từ bản Icehouse thì Keystone đã có thể sử dụng các liên kết xác thực thông qua module Apache cho các Identity Providers khác nhau.
      - Cơ bản thì Keystone sẽ sử dụng một bên thứ 3 để xác thực, nó có thể là những phần mềm sử dụng các backends (LDAP, AD, MongoDB) hoặc mạng xã hội (Google, Facebook, Twitter).
      - Ưu nhược điểm:

      Ưu điểm:

      - Có thể tận dụng phần mềm và cơ sở hạ tầng cũ để xác thực cũng như lấy thông tin của users.
      - Tách biệt keystone và nhiệm vụ định danh, xác thực thông tin.
      - Mở ra cánh cửa mới cho những khả năng mới ví dụ như single signon và hybrid cloud
      - Keystone không thể xem mật khẩu, mọi thứ đều không còn liên quan tới keystone.

      Nhược điểm:

      - Phức tạp nhất về việc setup trong các loại.

   5. ##### Các trường hợp sử dụng trong thực tế cho các backends

      |   **Use Cases**   | **Identity Source**                                          |
      | :---------------: | :----------------------------------------------------------- |
      |        SQL        | \- Testing hoặc developing với OpenStack<br/>\- Ít users<br/>\- OpenStack specific accounts |
      |       LDAP        | \- Nếu có sẵn LDAP trong công ty<br/>\- Sử dụng một mình LDAP nếu bạn có thể tạo service accounts trong LDAP |
      | Multiple Backends | \- Được sử dụng nhiều trong các doanh nghiệp<br/>\- Dùng nếu service user không được cho phép trong LDAP |
      | Identity Provider | \- Nếu bạn muốn có những lợi ích từ cơ chế mới Federated Identity<br/>\- Nếu đã có sẵn identity provider<br/>\- Keystone không thể kết nối tới LDAP<br/>\- Non-LDAP identity source |

5. ## User management

   - Các dịch vụ riêng lẻ được gán nghĩa cho các role, từ đó giới hạn hoặc ủy quyền truy cập cho user với role đó đến các dịch vụ. Role được cấu hình trong file **policy.yaml** của mỗi dịch vụ.

   - Dịch vụ Identity sẽ gán một project và một role cho một user. Nghĩa là role này sẽ ủy quyền truy cập các dịch vụ cho user nhưng chỉ trong phạm vi của project được cấu hình.

   - User có thể có các role khác nhau trong các project khác nhau. Cũng có thể có nhiều role trong cùng một project.

   - File **/etc/[SERVICE_CODENAME]/policy.yaml** quản lý các tác vụ mà user có thể thực hiện cho service đó.

   - Mặc định file **policy.yaml** của các dịch vụ thường chỉ nhận ra role **admin**, bất kỳ user với bất kỳ role nào trong một project đều có thể truy cập mọi hoạt động mà không cần role admin.

     Để hạn chế quyền thực hiện các hành động trong, ví dụ, Compute Service, đầu tiên cần tạo một role trong dịch vụ Identity, gán role cho user sau đó sửa file /etc/nova/policy.yaml để cấu hình quyền truy cập compute cho role này.

6. ## Authentication

   Có nhiều các để xác thực với keystone nhưng có hai cách phổ biến nhất là dùng password và dùng token.

   1. #### Password

      ![img](https://i.imgur.com/08xTI0J.png)

   2. #### Token

      ![img](https://i.imgur.com/hOorgcA.png)

7. ## Keystone workflow

   ![img](https://i.imgur.com/8lgwq29.png)

   