# Install
## 1. Cài đặt và sử dụng Nginx in CentOs7 - Online
### Bước 1: Cài đặt Nginx
- Để cài Nginx trên CentOS, chúng ta sẽ cần thêm EPEL repository giúp tạo, duy trì và quản lý các gói bổ sung.
```
sudo yum install epel-release -y
```
- Cài đặt và khởi động Nginx vào server
```
sudo yum install nginx
```
### Bước 2: Điều chỉnh Firewall
Bạn cần phải cấu hình tường lửa để Nginx có thể đáp ứng dịch vụ qua internet. Thông thường CentOS 7 sẽ mặc định chặn truy cập vào port 80 và 443, điều này sẽ trực tiếp chặn các traffic của Nginx. Để cho phép các traffic HTTP và HTTPS ta thực thi lần lượt các lệnh sau:

```
sudo firewall-cmd --permanent --zone=public --add-service=https
sudo firewall-cmd --reload
```

### Bước 3: Kiểm tra web server

```
systemctl status nginx
```

### Bước 4: Quản lý dịch vụ Nginx
Bởi vì Nginx được quản lý bằng systemd cho nên muốn điều chỉnh bạn phải sử dụng công cụ systemctl. Bây giờ chúng ta sẽ đi qua các lệnh quản lý cơ bản:
- Khởi động Web Server
```
sudo systemctl start nginx
```

- Dừng Web Server
```
sudo systemctl stop nginx
```

- Restart Web Server
```
sudo systemctl restart nginx
```

- Reload Web Server mà không mất kết nối: Thường được sử dụng trong trường hợp bạn thay đổi cấu hình và muốn Nginx cập nhật cấu hình vừa thay đổi.
```
sudo systemctl reload nginx
```

- Bật khởi động cùng hệ thống: Thao tác này sẽ giúp Nginx tự khởi động cùng hệ thống sau khi bạn khởi động lại máy chủ.
```
sudo systemctl enable nginx
```

- Tắt khởi động cùng hệ thống
```
sudo systemctl disable nginx
```

- Kiểm tra tình trạng của web server
```
 sudo systemctl status nginx
```
### Bước 5: Cấu hình server block (virtual host)
Server block trong Nginx tương tự như Virtualhost trong Apache. Server block giúp bạn khai báo và chạy nhiều website với nhiều tên miền khác nhau trên cùng một máy chủ Nginx.
Trong các thao tác dưới đây mình sẽ sử dụng tên miền your_domain.com để tạo nên một website mới.
- 1. Tạo thư mực /var/www/your_domain.com/html để chứa mã nguồn website.
```
sudo mkdir -p /var/www/your_domain.com/html
```

- 2. Trao quyền và chủ sở hữu cho thư mục.
```
sudo chown –R $USER:$USER /var/www/your_domain.com
sudo chmod –R 755 /var/www
```
- 3. Tạo một file index.html tại /var/www/your_domain.com/html/ chứa nội dung sau:
```
<html>
    <head>
        <title>Server cua toi</title>
    </head>
    <body>
        <h1>Xin chao the gioi</h1>
    </body>
</html>
```

- 4. Tạo một file config cho website your_domain.com
```
sudo nano /etc/nginx/conf.d/your_domain.com.conf
```
Nội dung dưới đây vào:
```
server {
    listen 80;
    listen [::]:80;

    root /var/www/your_domain.com/html;

    index index.html;

    server_name your_domain.com www.your_domain.com;

    access_log /var/log/nginx/your_domain.com.access.log;
    error_log /var/log/nginx/your_domain.com.error.log;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

- 5. Sau khi lưu lại file, bạn sẽ kiểm tra config có đúng syntax hay không bằng:
```
sudo nginx -t
```

Nếu mọi thứ đều ổn thì bạn sẽ nhận output như sau:
```
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

- 6. Khởi động lại Web Server bằng lệnh mình đã giới thiệu ở Bước 4: Quản lý dịch vụ Nginx.
```
sudo systemctl restart nginx
```

- 7. Mở trình duyệt truy cập vào website http://your_domain.com để xem kết quả. Nếu ở bước này bạn không có tên miền, bạn có thể trỏ file hosts trên máy tính của mình để truy cập.

### Bước 6: Các thư mục quan trọng trong Nginx
Bạn cần phải nắm các thư mục và các file cấu hình quan trọng của Nginx. Cụ thể:

1. Nội dung mã nguồn website:
```
/var/www/html/ – Đây là thư mục chứa nội dung mã nguồn website.
```
2. Các thư mục và file cấu hình của Nginx:

```
/etc/nginx/ – Thư mục chứa các file cấu hình của Nginx
/etc/nginx/nginx.conf – File config chính Nginx.
/etc/nginx/sites-available/ – Thư mục chứa các file cấu hình server block.
/etc/nginx/sites-enabled/ – Thư mục chứa Danh sách các server blocks được kích hoạt.
```

3. Các file log của Nginx, bao gồm access log và error log:

```
/var/log/nginx/access.log – Chứa các lịch sử request tới Web server của bạn (Bạn có thể thay đổi việc lưu log lại hay không).
/var/log/ngins/error.log – Chứa các lỗi từ Nginx.
```

## 2. Cài đặt Nginx theo source
- 1. Cài đặt các package cần thiết để compile Nginx từ source
```
yum groupinstall " Development Tools" -y
yum install zlib-devel pcre-devel openssl-devel wget -y
yum install epel-release -y
```

- 2. Cài đặt thêm các thành phần phụ thuộc của nginx

```
yum install perl perl-devel perl-ExtUtils-Embed libxslt libxslt-devel libxml2 libxml2-devel gd gd-devel GeoIP GeoIP-devel -y
```

- 3. Tiến hành download source nginx tại trang https://nginx.org/download/. Trong bài viết này sử dụng source version nginx 1.15.0
```
cd /usr/src/
wget https://nginx.org/download/nginx-1.15.0.tar.gz
tar -xzf nginx-1.15.0.tar.gz
```
Truy cập vào đường dẫn chứa source nginx vừa giải nén
```
cd nginx-1.15.0/
```
Tiến hành config từ script

```
./configure --prefix=/etc/nginx --sbin-path=/usr/sbin/nginx --error-log-path=/var/log/nginx/error.log --pid-path=/var/run/nginx.pid --lock-path=/var/run/nginx.lock --user=nginx --group=nginx

make
make install
```

- 4. Tạo user và tiến hành phân quyền owner cho thư mục
```
useradd nginx
chown -R nginx:nginx /etc/nginx/
```

- 5. Tạo file để chạy lệnh mỗi khi stop hoặc start service nginx

```

cat >> /usr/lib/systemd/system/nginx.service << "EOF"
[Unit]
Description=nginx - high performance web server
Documentation=https://nginx.org/en/docs/
After=network-online.target remote-fs.target nss-lookup.target
Wants=network-online.target

[Service]
Type=forking
PIDFile=/var/run/nginx.pid
ExecStartPre=/usr/sbin/nginx -t -c /etc/nginx/conf/nginx.conf
ExecStart=/usr/sbin/nginx -c /etc/nginx/conf/nginx.conf
ExecReload=/bin/kill -s HUP $MAINPID
ExecStop=/bin/kill -s TERM $MAINPID

[Install]
WantedBy=multi-user.target
EOF
```

Start Service nginx và cấu hình auto start service mỗi khi server reboot systemctl start nginx systemctl enable nginx Chuyển tới bước tiếp theo

- 6. Kiểm tra lại cấu hình nginx xem đã chính xác chưa bằng lệnh bên dưới Với kiểu cài đặt theo lệnh
```
yum nginx -t -c /etc/nginx/nginx.conf
```
Với kiều cài đặt theo

```
Source nginx -t -c /etc/nginx/conf/nginx.conf
```
