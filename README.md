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

## 3.4. 

## 3.5. 

# <p align="center">*--- THE END ---*</p>
