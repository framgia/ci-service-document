# Framgia CI Service Document

## Framgia CI Service
**Framgia CI Service** bao gồm 4 dịch vụ chính, đó là:
- **Framgia Drone Service**: Là công cụ CI/CD open source, được viết bằng Golang, hoạt động dựa vào Docker.
Framgia Drone sử dụng phiên bản Drone **0.4.2**. Về document của Drone, bạn có thể tham khảo ở [đây](http://readme.drone.io/0.4/usage/overview/)
- **Framgia CI Report Service**: Là hệ thống do Framgia phát triển, dùng để lưu và hiển thị test reports, cũng như theo dõi tình trạng các bản build.
- **Framgia CI CLI**: Một Command Line Interface được viết bằng python, có nhiệm vụ gọi các API của hệ thống Framgia CI Report,
cũng như để chạy các câu lệnh trong bản build được dễ dàng hơn. Cách thức cài đặt cũng như hướng dẫn về Framgia CI CLI có thể xem ở https://github.com/framgia/ci-report-tool/
- **Framgia GitHub Comment Service**: Là hệ thống phân tích reports và gửi comment lên Github. Chỉ hoạt động khi sử dụng cùng hệ thồng Framgia CI Report.
(Đương nhiên rồi, bạn cần phải có reports thì mới gửi được comment lên GitHub chứ :smile:)

## Framgia Drone Service
- URL: http://ci.framgia.vn/
- Requirement: Bạn cần liên hện với Admin để tạo tài khoản trên http://ci.framgia.vn/ . Ngoài ra, bạn cần có quyền admin với **Repository** trên GitHub để có thể active được repository đó.
- Framgia Drone Service có thể chạy độc lập mà không cần đến 3 thành phần còn lại. Tuy nhiên trong trường hợp đó, bạn chỉ có thể xác định được trạng thái của bản build trên GitHub. Nếu bạn muốn lưu trữ và xem kỹ report của từng bản build của mình, bạn cần sử dụng đến các công cụ khác trong hệ thống **Framgia CI Service**
- Sau khi active project trên Framgia Drone, bạn cần có một file config để báo cho Drone biết nó phải làm những gì. File này có tên `.drone.yml`, được viết bằng cú pháp `yaml`. Bạn có thể tìm hiểu kỹ hơn ở trang [document](http://readme.drone.io/0.4) của Drone. <br>
Ví dụ về file config cho một project Rails:
```
build:
  image: framgia/ruby-workspace-ci
  commands:
    - curl -o /usr/bin/framgia-ci https://raw.githubusercontent.com/framgia/ci-report-tool/master/dist/framgia-ci && chmod +x /usr/bin/framgia-ci
    - bundle install --path=/drone/.bundle
    - framgia-ci run
cache:
  mount:
    - .git
    - /drone/.bundle
```

Có thể thấy project này được build trên image Docker có tên là `framgia/ruby-workspace-ci`. Nội dung của image này các bạn có thể xem ở **Docker Hub** tại địa chỉ https://hub.docker.com/r/framgia/ruby-workspace-ci/

Về nội dung của phần config thì bao gồm 2 phần, phần `build` và phần `cache`. Phần build làm công việc đó là tải về `framgia-ci` tool, cài đặt các gem Ruby cần thiết từ folder cache đã được restore trước đó. Sau đó dùng công cụ `framgia-ci` để chạy test cũng như gửi report.

Kết thúc là quá trình cache lại thư mục `.git` và `/drone/.bundle` để những bản build sau có thể sử dụng được.

- Framgia Drone có sẵn nhiều **plugins** riêng biệt (không phải là plugins có sẵn của Drone), để phục vụ cho việc auto deployment với nhiều tools khác nhau (như **capistrano**, **rocketeer**, **ansible** ...), cũng như để notify lên Chatwork. Danh sách các plugins này có thể xem tại Docker Hub của Framgia CI Team: https://hub.docker.com/r/fdplugins/

- **Chú ý**:
    - Bạn nên tạo Docker Image cho project của mình, với đầy đủ các công cụ cần thiết ở trên đó để quá trình build được diễn ra nhanh hơn (không phải cài đặt những công cụ, packages khác)
    - Bạn nên sử dụng chức năng cache để việc cài đặt `bundle`, `npm`, `composer` diễn ra được nhanh hơn.
    - Ở phiên bản hiện tại của Drone, một Pull Request chỉ có thể restore cache từ bản cache của defaul branch. Vấn đề này sẽ được khắc phục khi Framgia Drone được nâng cấp lên bản mới.

- Bạn có thể tham khảo thêm về config ví dụ cho project PHP [ở đây](./php/), ví dụ cho project Ruby [ở đây](./ruby), hay ví dụ cho project Android [ở đây](./android).

## Framgia CI Report Service
- URL: http://ci-reports.framgia.vn/
- Requirement: Bạn cần có repository được active bên Framgia Drone để có thể theo dõi được trạng thái của chúng ở bên Framgia CI Report
- Hiện Framgia CI Report Service support 3 loại project cho Ruby, PHP và Android, với các loại test tool sau:
    - Ruby: `bundle-audit`, `rspec` with code coverage, `brakeman`, `reek`, `rubocop`, `rails_best_practices`
    - PHP: `phpcpd` (PHP Copy/Paste Detector), `phpmd` (PHP Mess Detector), `pdepend` (PHP Depend), `phpmetrics`, `phpcs` (PHP CodeSniffer), `phpunit` with Code Coverage
    - Android: `android-lint`, `checkstyle`, `findbugs`, `pmd`)
    - Ngoài ra còn support hiển thị Javascript & CSS (`eslint`, `scss-lint`) cho các project PHP và Ruby.
- Framgia CI Report cũng có các tính năng về notification lên Chatwork hay qua Email. Bạn cần có quyền admin với GitHub Repository để có thể chỉnh sửa các setting này trên Framgia CI Report Service.
- Để nhận thông báo về các bản build qua **Chatwork**, bạn có thể sử dụng Chatwork Bot của riêng mình, hoặc sử dụng bot mặc định mà Framgia CI cung cấp. Hãy add contact với [Framgia CI Bot](https://www.chatwork.com/framgia-ci-bot) rồi add nó vào box Chatwork mà bạn mong muốn nhận message, và Framgia CI Service sẽ thực hiện các công việc còn lại cho bạn.

## Framgia CI CLI tool
- URL: https://github.com/framgia/ci-report-tool/
- Nếu bạn dùng Mac, bạn cần python 3.5 để cài đặt và sử dụng `framgia-ci` tool. Nếu bạn dùng Linux, bạn có thể cài thông qua `pip` giống Mac, hoặc là tải về file pre-compiled.
- Bạn cần một file config riêng cho Framgia CI CLI, nó có tên là `.framgia-ci.yml`. File này có thể được sinh bằng câu lệnh `framgia-ci init {project_type}`. Ví dụ:
```
framgia-ci init php
framgia-ci init ruby
framgia-ci init android
```
- Các file reports được sinh ra bắt buộc phải để trong một thư mục có tên `.framgia-ci-reports` để bên Framgia CI Report Service có thể nhận dạng được.

## Framgia GitHub Comment service
- Hệ thống Framgia GitHub Comment sẽ tự động được chạy sau khi Framgia CI Report Service hoàn thành việc copy các file reports từ Framgia Drone. Bạn hoàn toàn không cần thực hiện bất kỳ config nào cho việc này.
