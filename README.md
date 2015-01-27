# fluentd-guide
Hướng dẫn về Fluentd trong Ruby

  Đối với web service thì một phần quan trọng là log data của web service. Trong bài viết này sẽ hướng dẫn về Fluentd, một công cụ mạnh để xử lý log trong web service.
  
  1. Fluentd
    Fluentd là một ứng dụng hoàn toàn miễn phí và open-source giúp bạn có thể lấy được log ở web server và xử lý     log data đó. Bao gồm như lưu lên S3, lưu vào MongoDB hay chạy file ruby để xử lý dữ liệu đó. Trang chủ về          Fluentd:
  http://www.fluentd.org/

  2. Cài đặt Fluentd trong Ruby
    Cài đặt Fluentd trên Ruby rất đơn giản chỉ bằng cài đặt gem
    ```
    gem install fluentd --no-ri --no-rdoc
    ```
    Chạy dòng lệnh dưới đây để kiểm tra Fluentd
    ```
    $ fluentd --setup ./fluent
    $ fluentd -c ./fluent/fluent.conf -vv &
    $ echo '{"json":"message"}' | fluent-cat debug.test
    ```
    Trong file output in ra nếu in như dưới đây là Fluentd đã được cài đặt thành công
    ```
    2011-07-10 16:49:50 +0900 debug.test: {"json":"message"}
    ```
    
  3. Cơ bản trong Fluentd
    
