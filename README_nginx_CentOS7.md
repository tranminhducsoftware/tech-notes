# 🚀 Hướng Dẫn Cài Đặt Nginx Trên CentOS 7

Hướng dẫn từng bước để cài đặt và cấu hình Nginx trên CentOS 7, bao gồm cả cách cài qua `yum` và build từ source.

---

## ✅ 1. Cài Đặt Nginx Qua YUM (Online)

### 🔧 Bước 1: Cài EPEL và Nginx
```bash
sudo yum install epel-release -y
sudo yum install nginx -y
```

### 🔥 Bước 2: Mở Firewall cho HTTP/HTTPS
```bash
sudo firewall-cmd --permanent --zone=public --add-service=https
sudo firewall-cmd --reload
```

### 🔍 Bước 3: Kiểm Tra Trạng Thái Nginx
```bash
systemctl status nginx
```

### 📦 Bước 4: Quản Lý Dịch Vụ Nginx
```bash
sudo systemctl start nginx         # Khởi động
sudo systemctl stop nginx          # Dừng
sudo systemctl restart nginx       # Khởi động lại
sudo systemctl reload nginx        # Reload cấu hình
sudo systemctl enable nginx        # Tự động khởi động cùng hệ thống
sudo systemctl disable nginx       # Tắt tự động khởi động
sudo systemctl status nginx        # Kiểm tra trạng thái
```

### 🌐 Bước 5: Cấu Hình Server Block (Virtual Host)
```bash
# Tạo thư mục chứa mã nguồn
sudo mkdir -p /var/www/your_domain.com/html

# Cấp quyền
sudo chown -R $USER:$USER /var/www/your_domain.com
sudo chmod -R 755 /var/www

# Tạo index.html mẫu
echo "<h1>Xin chào thế giới</h1>" > /var/www/your_domain.com/html/index.html

# Tạo config virtual host
sudo nano /etc/nginx/conf.d/your_domain.com.conf
```

**Nội dung cấu hình mẫu:**
```nginx
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

**Kiểm tra config và restart Nginx:**
```bash
sudo nginx -t
sudo systemctl restart nginx
```

### 📁 Bước 6: Các Thư Mục Quan Trọng Trong Nginx

| Loại | Đường Dẫn |
|------|-----------|
| Website code | `/var/www/html/` |
| Cấu hình | `/etc/nginx/nginx.conf`, `/etc/nginx/conf.d/`, `/etc/nginx/sites-available/` |
| Log | `/var/log/nginx/access.log`, `/var/log/nginx/error.log` |

---

## 🛠 2. Cài Đặt Nginx Từ Source

### ⚙️ Bước 1: Cài Gói Phụ Thuộc
```bash
yum groupinstall "Development Tools" -y
yum install epel-release zlib-devel pcre-devel openssl-devel wget -y
yum install perl perl-devel perl-ExtUtils-Embed libxslt libxslt-devel libxml2 libxml2-devel gd gd-devel GeoIP GeoIP-devel -y
```

### 📥 Bước 2: Tải Source Nginx
```bash
cd /usr/src/
wget https://nginx.org/download/nginx-1.15.0.tar.gz
tar -xzf nginx-1.15.0.tar.gz
cd nginx-1.15.0/
```

### 🧱 Bước 3: Biên Dịch và Cài Đặt
```bash
./configure --prefix=/etc/nginx --sbin-path=/usr/sbin/nginx --error-log-path=/var/log/nginx/error.log --pid-path=/var/run/nginx.pid --lock-path=/var/run/nginx.lock --user=nginx --group=nginx

make
make install
```

### 👤 Bước 4: Tạo User và Cấp Quyền
```bash
useradd nginx
chown -R nginx:nginx /etc/nginx/
```

### 🧩 Bước 5: Tạo File Systemd cho Service
```bash
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

### 🚀 Bước 6: Khởi Động và Kiểm Tra
```bash
systemctl start nginx
systemctl enable nginx

# Kiểm tra cấu hình
nginx -t -c /etc/nginx/conf/nginx.conf
```

---

> ✅ Hướng dẫn chi tiết và chuyên nghiệp giúp bạn triển khai Nginx dễ dàng trên CentOS 7. Thích hợp dùng cho cả production và môi trường học tập.