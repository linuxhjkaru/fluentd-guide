# fluentd-guide
Hướng dẫn về Fluentd trong Ruby

  Đối với web service thì một phần quan trọng là log data của web service. Trong bài viết này sẽ hướng dẫn về Fluentd, một công cụ mạnh để xử lý log trong web service. Vì người sử dụng chỉ biết về Ruby nên sẽ có 1 phần hướng dẫn sâu hơn 1 chút về Fluentd trong Ruby on Rails
  
  1. Fluentd
  
    Fluentd là một ứng dụng hoàn toàn miễn phí và open-source giúp bạn có thể lấy được log ở web server và xử lý    log data đó. Dữ liệu log lấy đươc rất có thể từ nginx, apache hoặc đơn giản là từ các gói tin tcp hay các         message của http. Phần xử lý dữ liệu ao gồm như lưu lên S3, lưu vào MongoDB, Redis hay đơn giản là chạy file      ruby để xử lý dữ liệu đó. Trang chủ về Fluentd:
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
  
    Flow thực hiện của Fluentd đơn giản như sau: với mỗi một Log Data sẽ được gắn 1 tag, tag này tùy theo bản thân   chọn. Ví dụ có thể chọn tag: tag_s3, tag_mongodb, tag_exec... Các log tùy theo tag được gắn sẽ đươc xử lý theo    thiết lập trong file  td-agent.conf. Ví dụ log có tag là tag_s3 sẽ đươc lưu trên s3, hay log data có tag là       tag_mongodb sẽ được lưu trong mongodb tùy theo xử lý mà ta viết trong file td-agent.conf. Do đó điều quan         trọng nhất trong fluentd là viết các xử lý trong file td-agent.conf.
  
    Trong 1 file config thường sẽ có 4 thành phần là: source, match, include, system. Trong đó source và match là   2 thành phần bắt buộc phải có. inlude và system có thể có có thể không. Dưới đây sẽ hướng dẫn cụ thể về source    và match trong fluentd.

  3.1 Source
  
    Source sẽ là nơi xác định xem log data sẽ được lấy từ đâu. Trong Fluentd thì source sẽ bao gồm http và          forward. http sẽ lấy log từ các message là http, trong khi đó forward sẽ lấy log từ các gói tin tcp. Và dĩ nhiên   là trong config có thẻ thiết lập để lấy cả 2 loại cùng lúc. Ví dụ cho thiết lập từ gói tin tcp và http như sau
  ```
  <source>
    type forward
    port 24224
  </source>
  
  và cho http
  
  <source>
    type http
    port 9880
  </source>
  ```
  
  Phần thiết lập cho source đơn giản chỉ có như vậy.
  
  3.2 Match
  
    Đây là 1 phần rất quan trọng trong FLuentd khi sẽ quyết định xem log data mà nhận được sẽ xử lý như thế nào.     Lưu trên S3, lưu trong Mongodb hay đơn giản là lưu vào 1 file nào đó. Ví dụ cụ thể cho match
  
  ```
  <match myapp.access>
    type file
    path /var/log/fluent/access
  </match>
  ```
  
    Ở ví dụ trên thì tất cả các log data có tag là myapp-access sẽ đươc lưu vào file là                 /var/log/fluent/access.%Y-%m-%d. Ở phần match thì có 1 vài điểm cần lưu ý nhuư sau
    
    - Match sẽ có phần đuôi mở rộng bao gồm
      + a.* sẽ chỉ match những log data có tag là a.b ko nhận những tag là a.b.c
      + a.** sẽ match nhũng log data có là a.b và cả a.b.c
      + {a,b} sẽ math những log data có tag là a và b
    - 1 log sẽ được quét từ đầu file Log đến cuối file log. Nếu match ở tag nào thì sẽ thực hiện ở tag đó và bỏ       qua tag tiếp theo. Do đó cần phải rất cẩn thận khi thiết lập file config. Vi dụ cho trường hợp sau
    
    ```
    <match myapp.*>
      type file
      path /var/log/fluent/access
    </match>
    <match myapp.test>
      type exec_filter
      command /usr/local/rbenv/shims/ruby /ruby.rb
    </match>
    ```
    
    Trong ví dụ trên thì theo như chúng ta kì vọng là 1 log data có tag là myapp.test sẽ được coi là input của      file ruby.rb và thực hiện file đó. Tuy nhiên vì trước đấy chúng ta đã khai báo 1 match là myapp.* cho nên log     data này sẽ thực hiện match myapp.* và bỏ qua phần match phía sau. Do đó cần hết sức lưu ý khi viết các match     cho file fluent.conf .
    
    Một điểm cần hết sức chú ý trong phần match đó chính là thiết lập type trong phần match. Một vài loại type mà   chúng ta cần chú ý như sau:
    
    + exec_filter: dùng để thực hiện 1 file bên ngoài với input là log data nhận đươc. Ở ví dụ trên là file         ruby.rb
    + file: Log nhận được sẽ được lưu vào 1 file mà có phần path chỉ định
    + mongo: Log nhận được sẽ được lưu vào mongodb ( ví dụ bên dưới)
    + s3: Log nhận được sẽ được lưu lên S3 ( ví dụ bên dưới)
    
    
    
  3.3 Include và System
  
    Include chủ yếu được sử dụng cho trường hợp muốn include các file config khác. Còn System được sử dụng để       thiết lập cho các config hệ thống. Phần này các bạn có thể xem trên doc của Fluentd.
    
  Với một file fluend.conf thì chỉ cần 2 phần là source và match bình thường là có thể chạy được phần xử lý log
  
  4. Một vài ví dụ về Fluend

    Trong các ví dụ dưới đây sẽ chỉ nêu 1 vài ví dụ về viết phần match, còn phần source thì dễ nên các bạn tự ngâm   cứu và viết.
  
 - Ví dụ 1: Log khi nhận được sẽ được chuyển lên S3 và lưu trên S3
 
  ```
  <match s3.client>
    type forest
    subtype s3
    remove_prefix s3.client
    <template>
      aws_key_id ****************
      aws_sec_key ********************
      s3_bucket test
  
      path log/__TAG__/
      buffer_path /var/log/td-agent/buffer/test/__TAG__
      
      time_slice_format %Y%m/%d/%Y%m%d_%H
      time_slice_wait 5m
      
      buffer_chunk_limit 256m
    </template>
  </match>
  ```
  
    1 điểm cần lưu ý là nên thiết lập buffer_chunk_limit trong trường hợp này vì khi lưu lên S3 thì không phải cứ   khi nào mà có log thì nó sẽ được lưu lên S3 mà vì trên S3 các dữ liêu log được lưu theo giờ nên cứ mỗi giờ thì    các dữ liệu mới được lueu trên S3. Do đó thiết lập buffer_chunk_limit là cần thiết.
    
  - Ví dụ 2: Log nhận được sẽ là input thực hiện 1 file ruby nào đó:
  
  ```
  <match exec.client>
    type exec_filter
    command /usr/local/rbenv/shims/ruby /var/td-agent/ruby.rb
    time_key got_at
    time_format %Y-%m-%d %H:%M:%S
    in_format json
    out_format msgpack
    buffer_queue_limit 32
    buffer_chunk_limit 4m
  </match>
  ```
  
    Trong ví dụ trên thì log data sẽ là input của file ruby.rb. Mỗi khi có input data thì file sẽ được thực hiện.
  Dữ liệu đầu vào sẽ là json còn dữ liệu đầu ra sử dụng msgpack (Thông tin về msgpack có thể tìm hiểu qua web:
  http://msgpack.org/ )
  
  - Ví dụ 3: Dữ liệu được lưu trong mongodb
  
  ```
  <match **>
    type mongo
    database <db name> #(required)
    collection <collection name> #(optional; default="untagged")
    host <hostname> #(optional; default="localhost")
    port <port> #(optional; default=27017)
  </match>
  ```
  
  Phần này không có gì để nói nhiều :))
  
  5. Fluentd trong Ruby on Rails
  
    Đối với RoR thì 1 gem nên sử dụng khi muốn cái đặt Fluend là ```gem 'act-fluent-logger-rails'``` Song song đó   là gem ```gem 'lograge'``` giúp cho việc dễ dàng sử dụng và đọc log data.
    Sau đó setting trong file config/application.rb như sau
 
  ```
  module SampleApp
    class Application < Rails::Application
      # other lines...
      config.log_level = :info
      config.logger = ActFluentLoggerRails::Logger.new
      config.lograge.enabled = true
      config.lograge.formatter = Lograge::Formatters::Json.new
    end
  end
  ```
  
  Và cuối cùng trong flie config/fluent-logger.yml chúng ta sẽ set tag cho log data như đã nói ở trên
  
  ```
  development:
  fluent_host:   '127.0.0.1'
  fluent_port:   24224
  tag:           'foo'
  messages_type: 'string'
  
  production:
    fluent_host:   '127.0.0.1'
    fluent_port:   24224
    tag:           'foo'
    messages_type: 'string'
  ```
  
  Sau khi thiết lập như trên thì log sau khi sử dụng fluentd của chúng ta có dạng như sau
  
  ```
  2014-07-07 19:39:01 +0000 foo: {"messages":"{\"method\":\"GET\",\"path\":\"/\",\"format\":\"*/*\",\"controller\":\"static_pages\",\"action\":\"home\",\"status\":200,\"duration\":550.14,\"view\":462.89,\"db\":1.2}","level":"INFO"}
  ```

    Thường thì với 1 RoR app thì thiết lập như vậy. Song trên thực tế với 1 RoR app chạy trên server sử dụng nginx   thì không cần phải sử dung nhiều gem cũng như phải thiết lập tag cho log data như vậy. Sử dụng Javascript gửi 1   request get đến server kèm theo tag. Sau đó xử lý trong nginx để các request này chuyển đến server xử lý log      data thì hoàn toàn có thể xử lý đươc log mà không cần những thiết lập như trên.
