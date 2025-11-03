# <p align="center">🌏 BÀI TẬP 3 - PHÁT TRIỂN ỨNG DỤNG TRÊN NỀN WEB</p>
**Giảng viên:** Đỗ Duy Cốp   
**Lớp học phần:** K58KTP   
**Sinh viên thực hiện:** Nguyễn Văn Thứ   
**MSSV:** K225480106062   
**Ngày giao:** 2025-10-24 13:50   
**Hạn nộp:** 2025-11-05 00:00   
**Đề tài:** Xây dựng một hệ thống **web IOT (Giám sát dữ liệu IOT)** dạng **Single Page Application (SPA)**, triển khai trên **Linux (Docker Desktop + WSL2 + Ubuntu)**.

---
# I. YÊU CẦU BÀI TẬP
💻 Bài tập yêu cầu xây dựng một hệ thống **web IOT (Giám sát dữ liệu IOT)** dạng **Single Page Application (SPA)**, triển khai trên **Linux (Docker Desktop + WSL2 + Ubuntu)**.

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
## 3.1. Kiểm tra trạng thái WSL2 + UBUNTU + DOCKER
- Kiểm tra WSL đã bật chưa bằng PowerShell với quyền Admin, chạy lệnh `wsl --status`. Kết quả WSL2 đã `enable thành công` và `Docker Desktop đang set làm default distribution`

- Kiểm tra Docker Desktop đang chạy bằng PowerShell với quyền Admin, chạy lệnh `docker --version`. kết quả đang chạy như hình.

- Test Docker bằng `hello-world` bằng PowerShell với quyền Admin, chạy lệnh `docker run hello-world`. Kết quả chạy thành công như hình.

# <p align="center">*--- THE END ---*</p>
