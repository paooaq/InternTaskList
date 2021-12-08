# Tìm hiểu về docker và container

1. ## Docker là gì?

   Docker là một nền tảng cho developers và sysadmin để develop, deploy và run application với container. Nó cho phép tạo các môi trường độc lập và tách biệt để khởi chạy và phát triển ứng dụng và môi trường này được gọi là container. Khi cần deploy lên bất kỳ server nào chỉ cần run container của Docker thì application của bạn sẽ được khởi chạy ngay lập tức.

2. ## Chức năng và vai trò của docker

   - Docker cho phép phát triển, di chuyển và chạy các ứng dụng dựa vào công nghệ ảo hóa trong Linux

   - Tự động triển khai các ứng dụng bên trong các container bằng cách cung cấp thêm một lớp trừu tượng và tự động hóa việc ảo hóa mức (OS)

   - Docker có thể được sử dụng trên các hệ điều hành như: Windows, Linux, MacOS

     => Docker giúp cho việc vận chuyển phần mềm nhanh hơn

     => Nhanh chóng triển khai, khởi động và di chuyển

     => Bảo mật và hỗ trợ APIs để giao tiếp với các container

     => Khắc phục sự cố dễ dàng hơn so với đóng vào container nhỏ

     => Chạy được nhiều phần mềm trên 1 máy -> tiết kiệm tài nguyên và tiền bạc

3. ## Một số khái niệm trong docker

   1. Dockerfile

      - Docker image có thể được tạo ra tự động bằng cách đọc các chỉ dẫn trong Dockerfile

      - Dockerfile mô tả ứng dụng và nó nói với docker cách để xây dựng nó thành 1 image

      - Dockerfile được đặt tên là `Dockerfile`, ngoài tên này ra các tên khác đều không hợp lệ

        ![img](https://hocchudong.com/wp-content/uploads/2021/04/image-44.png)

   2. Docker Image

      - Image trong docker còn được gọi là mirror, nó là 1 đơn vị đóng gói chứa mọi thứ cần thiết để 1 ứng dụng chạy

      - Có thể có được image docker bằng cách pulling từ image registry

      - Image được tạo thành từ nhiều layer xếp chồng lên nhau

        ![img](https://hocchudong.com/wp-content/uploads/2021/04/image-33.png)

   3. Registry

      - Docker Registry là nơi lưu trữ các image với hai chế độ là private và public

      - Cho phép chia sẻ các image template để sử dụng trong quá trình làm việc với Docker

      - Cơ quan registry phổ biến nhất là Docker Hub

        ![img](https://hocchudong.com/wp-content/uploads/2021/04/image-45.png)

   4. Docker container

      - Một image có thể được sử dụng để tạo 1 hay nhiều container

      - Có thể `create`, `start`, `stop`, `move` hoặc `delete` dựa trên Docker API hoặc Docker CLI

        ![img](https://github.com/hungviet99/ghichep_docker/raw/main/images/docker6.png)

      - Trong mô hình container, hệ điều hành yêu cầu tất cả tài nguyên phần cứng. Cài đặt container engine là docker. Công cụ container sẽ lấy các tài nguyên hệ điều hành và ghép chúng lại thành cấu trúc riêng gọi là container.

      - Mỗi container như 1 hệ điều hành thực sự, bên trong mỗi container sẽ chạy 1 ứng dụng

        ![img](https://github.com/hungviet99/ghichep_docker/raw/main/images/docker8.png)

      - Container và VM có sự cách ly và phân bổ tài nguyên tương tự nhưng có chức năng khác nhau vì container là ảo hóa hệ điều hành thay vì phần cứng. Các container có tính portable và hiệu quả hơn

        ![What is a Container? | App Containerization | Docker](https://www.docker.com/sites/default/files/d8/2018-11/docker-containerized-and-vm-transparent-bg.png)

      - Quá trình đưa ứng dụng chạy trong container được hiểu như sau:

        - Code App 

        - Tạo Dockerfile mô tả ứng dụng, các phụ thuộc và cách chạy ứng dụng đó

        - Build Dockerfile thành image

        - push image mới build vào registry

        - Chạy container từ image

          ![img](https://github.com/hungviet99/ghichep_docker/raw/main/images/docker9.png)

   5. Cài đặt docker

      - Gỡ bỏ phiên bản cũ của docker

        ```
        sudo apt-get remove docker docker-engine docker.io containerd runc
        ```

      - Cài đặt Docker Repository

        1. Cập nhật gói apt và cài đặt các gói để cho phép apt sử dụng kho lưu trữ qua HTTPS

           ```
            sudo apt-get update
            
            sudo apt-get install \
               ca-certificates \
               curl \
               gnupg \
               lsb-release

        2. Thêm khóa GPG chính thức của Docker:

           ```
            curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
           ```

      - Cài đặt docker

        ```
        sudo apt-get update
        sudo apt-get install docker-ce docker-ce-cli containerd.io
        ```

        Verify cài đặt bằng các run container hello world

        ```
        sudo docker run hello-world

   6. Dựng webserver bằng docker

      - Tạo thư mục

        ```
        mkdir docker

      - Tạo file Dockerfile 

        ```
        touch /docker/Dockerfile
        
        FROM ubuntu 
        ENV DEBIAN_FRONTEND noninteractive
        RUN apt update && apt install -y tcl
        RUN apt-get update 
        RUN apt-get install apache2 -y
        RUN apt-get install apache2-utils -y 
        RUN apt-get clean 
        EXPOSE 80 CMD [“apache2ctl”, “-D”, “FOREGROUND”]
        ```

        - Đầu tiên tạo hình ảnh của mình từ hình ảnh cơ sở Ubuntu.
        - 2 dòng tiếp theo để loại bỏ lựa chọn area trong quá trình cài
        - Tiếp theo, chúng ta sẽ sử dụng lệnh RUN để cập nhật tất cả các gói trên hệ thống Ubuntu.
        - Tiếp theo, chúng tôi sử dụng lệnh RUN để cài đặt apache2 trên hình ảnh của chúng tôi.
        - Tiếp theo, chúng tôi sử dụng lệnh RUN để cài đặt các gói apache2 tiện ích cần thiết trên hình ảnh của chúng tôi.
        - Tiếp theo, chúng tôi sử dụng lệnh RUN để xóa các tệp không cần thiết khỏi hệ thống.
        - Lệnh EXPOSE được sử dụng để hiển thị cổng 80 của Apache trong vùng chứa với máy chủ Docker.
        - Cuối cùng, lệnh CMD được sử dụng để chạy apache2 trong nền.

      - Chạy lệnh docker build để tạo tệp docker

        ```
        sudo docker build –t webserver . 
        ```

      -  Tệp máy chủ web đã được tạo, bây giờ là lúc tạo vùng chứa từ hình ảnh. Chúng ta có thể làm điều này bằng lệnh Docker run

        ```
        sudo docker run –d –p 1234:80 webserver 
        ```

        - Số cổng mà vùng chứa hiển thị là 80. Do đó với lệnh **–p** , chúng tôi đang ánh xạ cùng một số cổng với số cổng 1234 trên localhost của chúng tôi.
        - Các **-d** tùy chọn được sử dụng để chạy các thùng chứa trong chế độ tách ra. Điều này để vùng chứa có thể chạy ở chế độ nền.

      - Nếu bạn truy cập cổng 1234 của máy chủ Docker trong trình duyệt web của mình, bây giờ bạn sẽ thấy rằng Apache đã được thiết lập và đang chạy

