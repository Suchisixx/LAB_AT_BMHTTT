
# BÁO CÁO THỰC HÀNH LAB 7
## Xây dựng Chính sách Web Protection trên mô hình pfSense + Web Proxy

---

## 1. Thông tin sinh viên
- **Họ và tên:** Vũ Bá Lực
- **MSSV:** 1150080104
- **Lớp:** 11_ĐH_THMT
- **Học phần:** An toàn và Bảo mật Thông tin

---

## 2. Mục tiêu bài LAB
- Xây dựng kiến trúc bảo vệ Web tách biệt: pfSense làm Firewall/Gateway và Ubuntu (WEB-PROXY) chạy Squid + E2Guardian + ClamAV.
- Ép buộc người dùng (Client) đi qua Proxy, ngăn chặn tuyệt đối việc bypass (tắt proxy) bằng Firewall Rule.
- Xây dựng và kiểm thử các chính sách lọc nội dung: Hard block, Warning/Temporary bypass, Whitelist.
- Áp dụng quy tắc thời gian (Schedule), quét virus (ClamAV) và tùy biến trang chặn (Custom Block Page).

---

## 3. Môi trường & Công cụ

| Thành phần | Phiên bản / Cấu hình |
|------------|-----------|
| VirtualBox | 7.0.x |
| pfSense CE | 2.7.x (3 Adapter: WAN, LAN, DMZ) |
| Windows Server | 2022 Standard (DC01 - DNS Server) |
| Ubuntu Server | 24.04 LTS (WEB-PROXY - Squid + E2Guardian + ClamAV) |
| Ubuntu Server | 24.04 LTS (DMZ-WEB - Nginx) |
| Kali Linux | Rolling (Client) |

---

## 4. Kết quả kiểm thử 

| STT | Kịch bản | Mô tả | Kết quả |
|:---:|:-----------|:-------|:-------:|
| 1 | Baseline | Bật Proxy vào `normal.lab7.test`, tắt Proxy thì bị pfSense chặn | PASS |
| 2 | Geo-Sim | Truy cập `china.lab7.test`, `mideast.lab7.test` | PASS (Bị Hard Block) |
| 3 | Job Search | Truy cập `jobs.lab7.test` | PASS (Bị Hard Block) |
| 4 | Warning | Truy cập `weapons.lab7.test` | PASS (Hiện cảnh báo, có nút Acknowledge) |
| 5 | Education | Truy cập `education.lab7.test` | PASS (Được phép) |
| 6 | Entertainment | Truy cập `entertainment.lab7.test` theo Schedule 60 phút | PASS (Trong giờ vào được, ngoài giờ bị chặn) |
| 7 | Streaming | Truy cập `streaming.lab7.test` | PASS (Cảnh báo + Bypass tạm thời) |
| 8 | Specific Game | Truy cập `blockedgame.lab7.test` | PASS (Bị Hard Block) |
| 9 | Antivirus | Tải file EICAR từ `download.lab7.test` | PASS (Bị ClamAV chặn) |
| 10 | Size Limit | Tải file `100MB.bin` và `110MB.bin` | PASS (110MB bị chặn bởi Squid) |

---

## 5. Vấn đề gặp phải và cách khắc phục

> **Lỗi 1:** Máy ảo Ubuntu (WEB-PROXY) không có IP tĩnh, không ping được ra ngoài.
>
> **Khắc phục:** Cấu hình lại file Netplan (`/etc/netplan/00-installer-config.yaml`), đặt IP tĩnh `10.6.0.30`, Gateway `10.6.0.1`, DNS `10.6.0.10`, sau đó chạy `sudo netplan apply`.

> **Lỗi 2:** Lỗi DNS khi cài đặt (Temporary failure resolving 'archive.ubuntu.com').
>
> **Khắc phục:** Sửa file `/etc/resolv.conf` hoặc Netplan, trỏ DNS về `8.8.8.8` để tải gói, sau đó mở Rule trên pfSense cho phép vùng DMZ/LAN ra Internet (Port 53, 80, 443).

> **Lỗi 3:** Không chia sẻ được file từ máy thật vào máy ảo (Permission denied `/media/sf_...`).
>
> **Khắc phục:** Cài VirtualBox Guest Additions, sau đó thêm user vào nhóm `vboxsf` bằng lệnh `sudo usermod -aG vboxsf luc`, sau đó khởi động lại máy ảo.

> **Lỗi 4:** Lỗi kết nối Proxy (Connection timed out / Site not responding).
>
> **Khắc phục:** 
> - Kiểm tra lại card mạng của DMZ-WEB trong VirtualBox (phải cắm đúng vào Internal Network `lab6-dmz`).
> - Kiểm tra Rule trên pfSense (đảm bảo Rule Pass cho Proxy vào DMZ nằm TRÊN Rule Block All).
> - Xóa Cache của Squid: `sudo systemctl stop squid`, `sudo rm -rf /var/spool/squid/*`, `sudo squid -z`, `sudo systemctl start squid`.

> **Lỗi 5:** E2Guardian báo lỗi "Could not perform content scan".
>
> **Khắc phục:** ClamAV chưa cập nhật database hoặc dịch vụ bị treo. Tạm thời comment dòng `contentscanner` trong `e2guardian.conf` để tiếp tục test các tính năng khác, hoặc chạy `sudo freshclam` với DNS `8.8.8.8`.

> **Lỗi 6:** Không tìm thấy file `custom.html` để tùy biến trang Block.
>
> **Khắc phục:** E2Guardian tìm file template theo đường dẫn mặc định. Cần sửa đường dẫn tuyệt đối trong `e2guardianf1.conf` thành `/etc/e2guardian/custom.html` hoặc tạo file trong thư mục `/usr/share/e2guardian/languages/ukenglish/`.

---

## 6. Checklist nộp bài

- [x] Topology/VM và IP WEB-PROXY 10.6.0.30.
- [x] Alias và ruleset pfSense ép Client đi qua proxy.
- [x] Trang normal được phép và phép thử direct web bị block khi tắt proxy.
- [x] Hard block Geo-Sim và Job Search.
- [x] Warning + temporary bypass cho site cảnh báo.
- [x] Education được phép.
- [x] Schedule 60 phút và kết quả trong/ngoài schedule.
- [x] EICAR bị chặn bởi AV hoặc log scanner chứng minh detection.
- [x] Log E2Guardian và Firewall tương ứng.
- [ ] HTTPS inspection (Nếu lớp thực hiện - Phần mở rộng).

---
