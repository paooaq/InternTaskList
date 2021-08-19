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

