# <p align="center">🌏 BÀI TẬP 3 - PHÁT TRIỂN ỨNG DỤNG TRÊN NỀN WEB</p>
**Giảng viên:** Đỗ Duy Cốp   
**Lớp học phần:** K58KTP   
**Sinh viên thực hiện:** Nguyễn Văn Thứ   
**MSSV:** K225480106062   
**Ngày giao:** 2025-10-24 13:50   
**Hạn nộp:** 2025-11-05 00:00   

---
## 💻 Đề tài: Xây dựng một hệ thống web IOT (Giám sát dữ liệu IOT) dạng Single Page Application (SPA), triển khai trên Linux (Docker Desktop + WSL2 + Ubuntu).

# I. YÊU CẦU BÀI TẬP

**1. Cài đặt môi trường Linux**   

**2. Cài đặt Docker (nếu dùng docker desktop trên windows thì nó có ngay)**  

**3. Sử dụng 1 file docker-compose.yml để cài đặt các docker container sau:**   
- `mariadb (3306):` Cơ sở dữ liệu quan hệ phpMyAdmin.
- `phpmyadmin (8080):` Quản lý database qua web MariaDB.
- `nodered/node-red (1880):` Lập trình luồng dữ liệu IoT InfluxDB, Grafana.
- `influxdb (8086):` Lưu dữ liệu dạng thời gian kết nối Node-RED, Grafana.
- `grafana/grafana (3000):` Trực quan hóa dữ liệu kết nối InfluxDB, MariaDB.
- `nginx (80,443):` Reverse proxy / Web server, kết nối với các container web khác.

**4. Lập trình web frontend+backend:** Web IOT: Giám sát dữ liệu IOT.   
 - Tạo web dạng Single Page Application (SPA), chỉ gồm 1 file index.html, toàn bộ giao diện do javascript sinh động.
 - Có tính năng login, lưu phiên đăng nhập vào cookie và session
   Thông tin login lưu trong cơ sở dữ liệu của mariadb, được dev quản trị bằng phpmyadmin, yêu cầu sử dụng mã hoá khi gửi login.
   Chỉ cần login 1 lần, bao giờ logout thì mới phải login lại.
 - hiển thị giá trị mới nhất của các thông số đang giám sát, khi click vào thì hiển thị đồ thị lịch sử quá trình thay đổi (gọi grafana iframe để hiển thị)
 - backend: Sử dụng nodered để đọc dữ liệu từ các cảm biến (có thể dùng api online để lấy dữ liệu theo giời gian thực), 
   nodered sẽ lưu dữ liệu mới nhất (dạng update) vào cơ sở dữ liệu mariadb (sử dụng phpmyadmin để tạp table và quản trị lần đầu)
   nodered sẽ lưu dữ liệu (insert) vào influxdb để lưu giá trị lịch sử, để cho grafana dùng để hiển thị biểu đồ.
   
**5. Nginx làm web-server**   
 - Cấu hình nginx để chạy được website qua url http://nguyenvanthu.com 
 - Cấu hình nginx để http://nguyenvanthu.com/nodered truy cập vào nodered qua cổng 80, (dù nodered đang chạy ở port 1880)
 - Cấu hình nginx để http://nguyenvanthu.com/grafana truy cập vào grafana qua cổng 80, (dù grafana đang chạy ở port 3000)
# II. CẤU TRÚC BÀI TẬP
```
/home/nguyenvanthu/webspalinux/  
│
├── docker-compose.yml             # File chính khai báo toàn bộ container
│
├── nginx/
│   └── default.conf               # File cấu hình nginx (reverse proxy, domain)
│  
│
├── node-red/
│   ├── data/                     
│
├── mariadb/
│   ├── data/                      
│
├── influxdb/
│   └── data/                      
│
├── grafana/
│   ├── data/                      
│   └── config/
        └── grafana.ini            
├── phpmyadmin/                    
│
└── frontend/
    ├── index.html                 
    ├── js/
    │   ├── app.js                 
    │   ├── login.js              
    │   └── cart.js               
    ├── css/
    │   └── style.css
    └── assets/
        └── images/  
```
# III. TRIỂN KHAI BÀI TẬP
## 3.1. CẤU HÌNH CÀI ĐẶT MÔI TRƯỜNG LINUX
- Kích hoạt WSL và cài đặt Ubuntu mở PowerShell (Run as Administrator) chạy lệnh: `wsl --install` đồng thời tiến hành thiết lập username và password.
<img width="1104" height="640" alt="Ảnh chụp màn hình 2025-11-05 170335" src="https://github.com/user-attachments/assets/eddade69-6680-49a8-8e6b-a4795671f65c" />

  + Update Ubuntu và cài đạt một số tiện ích cơ bản bằng lệnh sau bằng tập lệnh sau:
```
sudo apt update && sudo apt upgrade -y
sudo apt install -y curl git vim build-essential ca-certificates
sudo apt install curl wget -y
```
<img width="1480" height="757" alt="Ảnh chụp màn hình 2025-11-05 172652" src="https://github.com/user-attachments/assets/c9f13f61-10bc-4509-ac4c-c8f18dea7a7a" />

- Kiểm tra Ubuntu bằng lệnh sau: `lsb_release -a`
<img width="1485" height="134" alt="Ảnh chụp màn hình 2025-11-05 172836" src="https://github.com/user-attachments/assets/132d341d-1f9d-4fbc-8d95-b1140365f8dd" />

## 3.2. CÀI ĐẶT DOCKER & DOCKER COMPOSE
- Mở trong Ubuntu chạy tập lệnh sau: cài chính thức từ Docker, tự động cài `docker-ce, docker-ce-cli, containerd & docker.sock.`
```
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
```

- Thêm user vào group docker để chạy Docker mà không cần sudo bằng lệnh sau: `sudo usermod -aG docker $USER`
- Sau đó hạy trong PowerShell (không cần admin) với lệnh sau: `wsl --shutdown` để thoát Ubuntu áp dụng thay đổi. Sau đó mở lại Ubuntu.
- Kiểm tra test Docker bằng lệnh sau: `docker run hello-world` cho ra kết quả như hình sau đã thấy "Hello from Docker!" là thành công.
<img width="1916" height="681" alt="Ảnh chụp màn hình 2025-11-05 175931" src="https://github.com/user-attachments/assets/783851fd-2574-4603-bdb2-c725630edcb0" />

- Cài Docker Compose (bản binary độc lập) và tiến hành thay đổi quyền thực thi cho file bằng tập lệnh sau trên Ubuntu:
```
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```

- Kiểm tra Docker bản binary bằng lệnh sau: `docker-compose --version` kết quả thành công như hình với Version v2.40.3.
<img width="1916" height="295" alt="Ảnh chụp màn hình 2025-11-05 180931" src="https://github.com/user-attachments/assets/5bf46d92-5cd0-4d53-b0d9-6e4b8b07ad50" />

## 3.3. CẤU HÌNH DOCKER COMPOSE
#### Mục tiêu là cài đặt 6 container: `mariadb (3306), phpmyadmin (8080), nodered/node-red (1880), influxdb (8086), grafana/grafana (3000), nginx (80,443)`
- `mariadb (3306):` Cơ sở dữ liệu quan hệ phpMyAdmin.
- `phpmyadmin (8080):` Quản lý database qua web MariaDB.
- `nodered/node-red (1880):` Lập trình luồng dữ liệu IoT InfluxDB, Grafana.
- `influxdb (8086):` Lưu dữ liệu dạng thời gian kết nối Node-RED, Grafana.
- `grafana/grafana (3000):` Trực quan hóa dữ liệu kết nối InfluxDB, MariaDB.
- `nginx (80,443):` Reverse proxy / Web server, kết nối với các container web khác.
#### Tạo file docker-compose.yml để cài đặt các docker container trên:
- Tạo thư mục và chuyển đến nó bằng tập lệnh sau trên Ubuntu:
```
mkdir webspalinux
cd webapplinux
```
- Tạo file docker-compose.yml trong thư mục `webspalinux` bằng lệnh sau: `nano docker-compose.yml`
```
services:
  mariadb:
    image: mariadb:10.11
    container_name: mariadb-thu
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: root123
      MYSQL_DATABASE: ShopQA
      MYSQL_USER: thu
      MYSQL_PASSWORD: thu123
    ports:
      - "3306:3306"
    volumes:
      - ./mariadb/data:/var/lib/mysql
    networks:
      - thu-network

  phpmyadmin:
    image: phpmyadmin:latest
    container_name: phpmyadmin-thu
    restart: always
    environment:
      PMA_HOST: mariadb        # ✅ Sửa lỗi quan trọng
      PMA_PORT: 3306
      MYSQL_ROOT_PASSWORD: root123
    ports:
      - "8080:80"
    depends_on:
      - mariadb
    networks:
      - thu-network

  influxdb:
    image: influxdb:2.7
    container_name: influxdb-thu
    restart: always
    environment:
      DOCKER_INFLUXDB_INIT_MODE: setup
      DOCKER_INFLUXDB_INIT_USERNAME: admin
      DOCKER_INFLUXDB_INIT_PASSWORD: admin123
      DOCKER_INFLUXDB_INIT_ORG: thu-org
      DOCKER_INFLUXDB_INIT_BUCKET: thu-bucket
      DOCKER_INFLUXDB_INIT_ADMIN_TOKEN: super-secret-token
    ports:
      - "8086:8086"
    volumes:
      - ./influxdb/data:/var/lib/influxdb2
    networks:
      - thu-network

  nodered:
    image: nodered/node-red:latest
    container_name: nodered-thu
    restart: always
    user: "1000:1000"
    ports:
      - "1880:1880"
    volumes:
      - ./node-red/data:/data      # ✅ Chỉ mount thư mục data
    networks:
      - thu-network
    depends_on:
      - mariadb
      - influxdb

  grafana:
    image: grafana/grafana:latest
    container_name: grafana-thu
    restart: always
    environment:
      GF_SECURITY_ADMIN_USER: admin
      GF_SECURITY_ADMIN_PASSWORD: admin123
      GF_SERVER_HTTP_PORT: 3000
      GF_SERVER_ROOT_URL: "%(protocol)s://localhost:3000/grafana/"
      GF_SERVER_SERVE_FROM_SUB_PATH: "true"
    ports:
      - "3000:3000"
    volumes:
      - ./grafana/data:/var/lib/grafana   # ✅ Mount đúng thư mục data
    depends_on:
      - influxdb
    networks:
      - thu-network

  nginx:
    image: nginx:latest
    container_name: nginx-thu
    restart: always
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/default.conf:/etc/nginx/conf.d/default.conf:ro
      - ./nginx/certs:/etc/nginx/certs:ro
      - ./frontend:/usr/share/nginx/html:ro
    depends_on:
      - nodered
      - grafana
    networks:
      - thu-network

networks:
  thu-network:
    driver: bridge
```
- Khởi động lại tất cả các container bằng cách chạy lệnh sau trong thư mục project trên Ubuntu: `docker compose up -d`
<img width="1917" height="211" alt="Ảnh chụp màn hình 2025-11-06 102733" src="https://github.com/user-attachments/assets/c8ba3f06-4d9c-487d-81df-186f33d42d95" />

- Kiểm tra các container đang hoạt động bằng lệnh: `docker ps`
<img width="1916" height="379" alt="Ảnh chụp màn hình 2025-11-06 102743" src="https://github.com/user-attachments/assets/b3cdef33-6fbc-4e05-aa5e-4eff525b12b9" />

#### Cấu hình nginx làm web-server
- Tạo 1 file cấu hình `default.conf` trong thư mục `nginx`.
```
server {
    listen 80;
    server_name localhost nguyenvanthu.com;

    location / {
        root /usr/share/nginx/html;
        try_files $uri $uri/ /index.html;
    }

    location /api/ {
        proxy_pass http://nodered-thu:1880/api/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }

    location /nodered/ {
        proxy_pass http://nodered-thu:1880/;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }

    location /grafana/ {
        proxy_pass http://grafana-thu:3000/;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

- Sau khi cấu hình mọi thứ đã ổn, thì sẽ thấy các container này chạy trên các cổng tương ứng như hình bên dưới.
<img width="1917" height="1017" alt="Ảnh chụp màn hình 2025-11-06 002709" src="https://github.com/user-attachments/assets/3759bdee-a0b9-428c-b91a-7d462a16d948" />

<img width="1916" height="1030" alt="Ảnh chụp màn hình 2025-11-06 104230" src="https://github.com/user-attachments/assets/1c9a4880-0088-4c26-93c5-f0315d289a13" />

<img width="1915" height="1023" alt="Ảnh chụp màn hình 2025-11-06 104355" src="https://github.com/user-attachments/assets/937c8ccc-3c81-4ae0-9920-d2cc5d5f6513" />

<img width="1914" height="1021" alt="Ảnh chụp màn hình 2025-11-06 104410" src="https://github.com/user-attachments/assets/5daa573f-ecf8-4091-ac25-056f48924809" />

- Sau khi cấu hình thành công nginx thì em đã demo thành công domain `nguyenvanthu.com` như hình.
<img width="1915" height="1079" alt="Ảnh chụp màn hình 2025-11-06 104602" src="https://github.com/user-attachments/assets/88f7aff6-d260-492d-8437-a1fc435b1504" />

- `Website chính:` 👉 http://nguyenvanthu.com
- `Node-RED:` 👉 http://nguyenvanthu.com/nodered
- `Grafana:` 👉 http://nguyenvanthu.com/grafana

## 3.4. LẬP TRÌNH WEB PRONTEND & BACKEND
#### Tạo 
- Vào phpMyAdmin → SQL → chạy. Xây dựng sơ sở dữ liệu
<img width="1911" height="1018" alt="Ảnh chụp màn hình 2025-11-06 222153" src="https://github.com/user-attachments/assets/66b3df5c-f3f3-4e5e-9784-45752a91104e" />

- Nodered test thử hệ thống
<img width="1917" height="560" alt="Ảnh chụp màn hình 2025-11-06 221649" src="https://github.com/user-attachments/assets/ec93581f-10ed-4978-be7a-da0f8951ef46" />

## 3.5. NGINX LÀM WEB-SERVER
- Giao diện Frontend của hệ thống.
<img width="1919" height="1020" alt="Ảnh chụp màn hình 2025-11-06 231836" src="https://github.com/user-attachments/assets/55bed2f1-bb79-4394-b569-f6a1c2a4f729" />

## 3.6. TỔNG KẾT
👉 Sau khi nghiên cứu và làm bài tập này, cá nhân em đã nhận thấy rằng:

- Việc cấu hình và cài đặt các Docker Container, Ubuntu trên môi trường Linux rất quan trọng cho cả hệ thống.
- Xây dựng Web IOT dưới dạng SPA đầy đủ frontend – backend – database – giám sát.
- Bài làm của em chưa được hoàn thiện vẫn đang tiếp tục thực hiện trong việc xây dựng Backend - Database - và giám sát hệ thống nhưng em cũng đã hiểu được một phần trong việc xây dựng và kết nối chúng để vẫn hành và duy trì. Em sẽ cố găng tiếp tục hoàn thiện các bước còn lại ạ!
 
# <p align="center">*--- THE END ---*</p>
