# ELK Stack
Setup and run
\
(tham khảo: https://www.digitalocean.com/)
(tham khảo: https://viblo.asia/)
\
\
Logging là phần không thể thiếu trong các hệ thống máy tính, máy chủ.
nó giúp ta lưu lại quá trình hoạt động của ứng dụng nhằm phân tích
hoạt động, kiểm tra lỗi phát sinh khi chạy ... 
\
\
Đối với những hệ thống lớn, file log có thể lớn tới nhiều GB, hay
mô hình hệ thống microservices thì file log được đặt phân tán ở nhiều
nơi. Vì vậy, để giúp chúng ta tập trung xử lý và phân tích log, các công
cụ hỗ trợ ra đời, nổi bật trong số đó là elk stack hay còn được gọi là 
elastic stack.

# ELK Stack là gì?
ELK là viết tắt của `Elasticsearch`, `Logstash` và `Kibana`. Trong đó:
* Elasticsearch: là một công cụ phân tích và tìm kiếm RESTful phân tán
được thiết kế để đáp ứng rất nhiều trường hợp sử dụng. Nó là nơi lưu
trữ dữ (JSON format) và phân tích dữ liệu mạnh mẽ cũng như khả năng mở rộng cao.
* Logstash: đây là một công cụ sử dụng để thu thập, xử lý log. Nhiệm
vụ chính của logstash là thu thập log sau đó chuyển vào elasticsearch.
* Kibana là một giao diện người dùng mở và miễn phí cho phép bạn trực
quan hóa dữ liệu từ Elasticsearch của mình.

# Cài đặt ELK Stack

`Lưu ý`: Khi cài đặt Elastic stack bạn phải sử dụng các stack cùng một 
phiên bản. Trong hướng dẫn này chúng ta sẽ sử dụng phiên bản mới nhất
của package 7.x là Elasticsearch 7.7.1, Kibana 7.7.1, Logstash 7.7.1 và
Filebeat 7.7.1.

`Yêu cầu`: Ubuntu 20.04.
\
Cài đặt `curl` nếu chưa có, bấm `ctrl`+`alt`+`t` để mở terminal và chạy lệnh sau:
```
sudo apt update
```
```
sudo apt install curl
```
\
`Bước 1`: Chuẩn bị cho tải và cài đặt ELK.
\
Tải về GPG key và thêm vào APT:
```
curl -fsSL https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
```
Thêm danh sách nguồn ELASTIC:
```
echo "deb https://artifacts.elastic.co/packages/7.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-7.x.list
```
Cập nhật danh sách package:
```
sudo apt update
```
\
`Bước 2`: Tải, cài đặt, cấu hình Elasticsearch.
\
Tải và cài đặt:
```
sudo apt install elasticsearch
```
Sau khi tải và cài đăt xong, file cấu hình `elasticsearch.yml` sẽ được tạo tự động và
đặt trong đường dẫn `/etc/elasticsearch/` chạy lệnh sau để sửa file cấu hình:
```
sudo nano /etc/elasticsearch/elasticsearch.yml
```
`Lưu ý`: file cấu hình của elasticsearch sử dụng định dạng YAML, có nghĩa là nó sử
dụng định dạng thụt dòng. Hãy chắc rằng bạn không thêm bất kỳ dấu cách thừa nào khi 
chỉnh sửa file.
\
File `elasticsearch.yml` cung cấp nhiều tùy chọn cài đặt như: cluster, node, paths,
memory, network, discovery, và gateway. Phần lớn đều đã được cài đặt theo mặc định,
bạn có thẻ thay đổi cài đặt nếu cần. Mặc định elasticsearch chạy ở cổng 9200.
\
Tại file `elasticsearch.yml` tìm đến dòng `#network.host: localhost` và loại bỏ comment
bằng cách xóa dấu `#` ở đầu dòng. Sau đó lưu lại và đóng file bằng cách nhấn `CTRL + X`,
sau đó nhấn `Y` và cuối cùng là `ENTER`.
Chạy Elasticsearch với `sysemctl`:
```
sudo systemctl start elasticsearch
```
Cho phép Elasticsearch chạy mỗi khi hệ thống khởi động:
```
sudo systemctl enable elasticsearch
```
Kiểm tra elasticsearch đang chạy:
```
curl -X GET "localhost:9200"
```
Bạn sẽ nhận kết quả trả về tương tự nếu elasticsearch chạy thành công:
```
{
  "name" : "Elasticsearch",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "qqhFHPigQ9e2lk-a7AvLNQ",
  "version" : {
    "number" : "7.7.1",
    "build_flavor" : "default",
    "build_type" : "deb",
    "build_hash" : "ef48eb35cf30adf4db14086e8aabd07ef6fb113f",
    "build_date" : "2020-03-26T06:34:37.794943Z",
    "build_snapshot" : false,
    "lucene_version" : "8.5.1",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}
```
\
`Bước 3`: Tải, cài đặt, cấu hình Kibana.
\
Tải và cài đặt:
```
sudo apt install kibana
```
Cho phép chạy khi khởi động và chạy Kibana:
```
sudo systemctl enable kibana
```
```
sudo systemctl start kibana
```
Mặc định Kibana chạy ở cổng 5601. khi khởi động xong kibana, mở trình duyệt và truy
cập vào `localhost:5601`. Nếu kibana chạy thành công thì sẽ hiển thị giao diện truy cập.
\
`Bước 4`: Tải, cài đặt, cấu hình Logstash.
\
Tải và cài đặt:
```
sudo apt install logstash
```
File cấu hình Logstash được đặt ở đường dẫn sau `/etc/logstash/conf.d`. Cấu hình
logstash pipeline yêu cầu có hai phần tử đó là `input` và `output` ngoài ra ta có tùy 
chọn thêm là `filter`. Chúng ta sẽ tạo một file cấu hình để sử dụng Filebeat làm đầu vào
(Chúng ta sẽ cài filebeat ở phần sau):
```
sudo nano /etc/logstash/conf.d/02-beats-input.conf
```
Chỉ định một `beats` để lắng nghe ở cổng 5044 cho việc nhận log. Thêm nội dung sau 
vào file và lưu lại:
```
input {
  beats {
    port => 5044
  }
}
```
Tiếp theo, ta tạo thêm một file cấu hình đầu ra:
```
sudo nano /etc/logstash/conf.d/30-elasticsearch-output.conf
```
Thêm nội dung sau vào file và lưu lại:
```
output {
  if [@metadata][pipeline] {
	elasticsearch {
  	hosts => ["localhost:9200"]
  	manage_template => false
  	index => "%{[@metadata][beat]}-%{[@metadata][version]}-%{+YYYY.MM.dd}"
  	pipeline => "%{[@metadata][pipeline]}"
	}
  } else {
	elasticsearch {
  	hosts => ["localhost:9200"]
  	manage_template => false
  	index => "%{[@metadata][beat]}-%{[@metadata][version]}-%{+YYYY.MM.dd}"
	}
  }
}
```
Kiểm tra cấu hình hợp lệ:
```
sudo -u logstash /usr/share/logstash/bin/logstash --path.settings /etc/logstash -t
```
Nếu không có lỗi sẽ hiển thị `Config Validation Result: OK. Exiting Logstash`
\
Cho phép chạy khi khởi động và chạy Logstash:
```
sudo systemctl enable logstash
```
```
sudo systemctl start logstash
```
\
`Bước 5`: Tải, cài đặt, cấu hình Filebeat.
\
Filebeat được sử dụng để thu thập và gửi file log đến logstash.
\
Tải và cài đặt Filebeat:
```
sudo apt install filebeat
```
Mặc định, Filebeat được cấu hình để gửi log đến Elasticsearch, chúng ta cần cấu hình 
lại để Filebeat gửi log đến Logstash (lưu ý dấu cách thừa khi chỉnh sửa):
```
sudo nano /etc/filebeat/filebeat.yml
```
Tìm đến dòng sau và comment lại bằng cách thêm `#` ở đầu dòng:
```
...
#output.elasticsearch:
  # Array of hosts to connect to.
  #hosts: ["localhost:9200"]
...
```
Tiếp theo tìm đến dòng sau và loại bỏ comment như sau:
```
output.logstash:
  # The Logstash hosts
  hosts: ["localhost:5044"]
```
Lưu lại và đóng file.
\
Chức năng của filebeat có thể mở rộng với filebeat modules. Ở đây chúng ta sử sụng
`system` module để dễ dàng hơn thu thập và phân tích logs được tạo bởi các hệ thống
logging Linux phổ biến.
/
Thêm nó bằng cách:
```
sudo filebeat modules enable system
```
Kiểm tra module hỗ trợ và đã thêm:
```
sudo filebeat modules list
```
Output có dạng:
```
Output
Enabled:
system

Disabled:
apache2
auditd
elasticsearch
icinga
iis
kafka
kibana
logstash
mongodb
mysql
nginx
osquery
postgresql
redis
traefik
...
```
Thiết lập filebeat pipeline và phân tích log trước khi gửi tới Logstash.
```
sudo filebeat setup --pipelines --modules system
```
Thêm index template vào Elasticsearch
```
sudo filebeat setup --index-management -E output.logstash.enabled=false -E 'output.elasticsearch.hosts=["localhost:9200"]'
```
Kết quả trả về thành công:
```
Output
Index setup finished.
```
Để kiểm tra phiên bản filebeat cần kết nối với elasticsearch để tải dashboards. tải dashboards khi Logstash đang chạy, chúng ta tắt Logstash output và bật elasticsearch output
```
sudo filebeat setup -E output.logstash.enabled=false -E output.elasticsearch.hosts=['localhost:9200'] -E setup.kibana.host=localhost:5601
```
Kết quả trả về thành công:
```
Output
Overwriting ILM policy is disabled. Set `setup.ilm.overwrite:true` for enabling.

Index setup finished.
Loading dashboards (Kibana must be running and reachable)
Loaded dashboards
Setting up ML using setup --machine-learning is going to be removed in 8.0.0. Please use the ML app instead.
See more: https://www.elastic.co/guide/en/elastic-stack-overview/current/xpack-ml.html
Loaded machine learning job configurations
Loaded Ingest pipelines
```
Chạy filebeat:
```
sudo systemctl start filebeat
sudo systemctl enable filebeat
```
Kiểm tra elasticsearch có nhận được data từ filebeat hay không:
```
curl -XGET 'http://localhost:9200/filebeat-*/_search?pretty'
```
Kết quả trả về thành công:
```
Output
...
{
{
  "took" : 4,
  "timed_out" : false,
  "_shards" : {
    "total" : 2,
    "successful" : 2,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 4040,
      "relation" : "eq"
    },
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "filebeat-7.7.1-2020.06.04",
        "_type" : "_doc",
        "_id" : "FiZLgXIB75I8Lxc9ewIH",
        "_score" : 1.0,
        "_source" : {
          "cloud" : {
            "provider" : "digitalocean",
            "instance" : {
              "id" : "194878454"
            },
            "region" : "nyc1"
          },
          "@timestamp" : "2020-06-04T21:45:03.995Z",
          "agent" : {
            "version" : "7.7.1",
            "type" : "filebeat",
            "ephemeral_id" : "cbcefb9a-8d15-4ce4-bad4-962a80371ec0",
            "hostname" : "june-ubuntu-20-04-elasticstack",
            "id" : "fbd5956f-12ab-4227-9782-f8f1a19b7f32"
          },


...
```
\
`Bước 6`: Sử dụng Kibana để xem log.
\
Dùng trình duyệt mở `localhost:5601`.
Nhấn vào `Discover` ở thanh menu bên cạnh. chúng ta có thể xem log được gửi từ filebeat ở dây.
\
\
`Luu y`: Chay cau lenh sau de chinh sua file log hoac thu muc chua file log de thu thap:
```
sudo nano /etc/filebeat/modules.d/system.yml
```
