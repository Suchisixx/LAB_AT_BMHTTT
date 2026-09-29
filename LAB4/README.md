
# LAB 4 — KHẢO SÁT VÀ ĐÁNH GIÁ BỀ MẶT MẠNG BẰNG NMAP

## 1. Thông tin sinh viên

| Thông tin | Giá trị |
|-----------|---------|
| Họ và tên | `Vũ Bá Lực` |
| MSSV | `1150080104` |
| Lớp | `11_ĐH_THMT` |
| Môn học | An toàn Hệ thống Thông tin |

---

## 2. Mục tiêu

- Cài đặt và sử dụng **Nmap** trên Kali Linux và Windows 11.
- Dựng mạng **VirtualBox Host-Only** an toàn.
- Thực hiện **host discovery**, **TCP/UDP scan**, **version/OS detection**, **NSE scripts**.
- Xuất kết quả và viết báo cáo kỹ thuật.

>  Bài lab chỉ dùng cho **học tập**. Chỉ quét máy ảo trong mạng Host-Only của chính sinh viên.

---

## 3. Môi trường

| Thành phần | Phiên bản |
|------------|-----------|
| Máy thật | Windows 11 64-bit |
| VirtualBox | `[Điền phiên bản]` |
| Kali Linux | Kali Rolling (VM) |
| Metasploitable 2 | VM mục tiêu |
| Nmap | 7.99 |

---


---

## 4. Cách dựng môi trường

1. Tạo **Host-Only Network** trong VirtualBox: `192.168.56.1/24`.
2. Gán **Adapter 1 = Host-only** cho cả Kali và Metasploitable 2.
3. Tắt NAT trên Kali trước khi quét.
4. Tạo snapshot `Before-LAB4` cho từng VM.
5. Lấy IP: `ip -br addr` (Kali), `/sbin/ifconfig` (Metasploitable).
6. Kiểm tra kết nối: `ping -c 4 192.168.56.102`.

---

## 5. Các tình huống đã thực hiện

| # | Tình huống | Lệnh chính | Kết quả |
|---|------------|------------|---------|
| 1 | Host Discovery | `sudo nmap -sn 192.168.56.0/24` | 3 host up |
| 2 | TCP Connect Scan | `nmap -sT 192.168.56.102` | 23 open |
| 3 | SYN Scan | `sudo nmap -sS 192.168.56.102` | 23 open |
| 4 | FIN/Xmas/NULL | `sudo nmap -sF/-sX/-sN` | open\|filtered |
| 5 | ACK Scan | `sudo nmap -sA 192.168.56.102` | 1000 unfiltered |
| 6 | UDP Scan | `sudo nmap -sU --top-ports 20` | 2 open / 11 closed / 7 open\|filtered |
| 7 | Version Detection | `sudo nmap -sV 192.168.56.102` | vsftpd 2.3.4, Apache 2.2.8... |
| 8 | OS Detection | `sudo nmap -O 192.168.56.102` | Linux 2.6.9 - 2.6.33 |
| 9 | Aggressive Scan | `sudo nmap -A 192.168.56.102` | Tổng hợp 4 thành phần |
| 10 | NSE SMB | `--script smb-os-discovery` | Samba 3.0.20-Debian |
| 11 | Xuất kết quả | `-oN/-oX/-oG` + `xsltproc` | 4 file |
| 12 | Before/After Hardening | Tắt telnet | Cổng 23: open → closed |

---

## 6. Kết quả PASS / FAIL

| Hạng mục | Kết quả |
|----------|---------|
| Cài Nmap trên Kali | PASS |
| Cài Nmap trên Windows | PASS |
| Tạo Host-Only Network | PASS |
| Cấu hình VM | PASS |
| Ping Kali → Metasploitable | PASS |
| Host Discovery | PASS |
| TCP Scans | PASS |
| UDP Scan | PASS |
| Version + OS Detection | PASS |
| NSE smb-os-discovery | PASS |
| NSE smb-vuln-ms17-010 | Không xác định (Samba Linux) |
| Xuất kết quả | PASS |
| Before/After Hardening | PASS |

---

## 7. Lỗi gặp phải và cách khắc phục

| Lỗi | Nguyên nhân | Cách khắc phục |
|-----|-------------|----------------|
| `ipconfig: command not found` | Thiếu `/sbin` trong PATH | Dùng `/sbin/ifconfig` |
| Gõ dư prompt `kali@kali:~$` | Nhầm prompt với lệnh | Chỉ gõ phần sau dấu `$` |
| Metasploitable không có IP | Chưa gán Host-Only | Gán lại Adapter 1, chạy `sudo dhclient eth0` |
| Nmap không hiện MAC Windows | Ping scan Layer 3 | Dùng `-PR` hoặc `ipconfig /all` |
| NSE ms17-010 không kết luận | Lỗ hổng Windows, không áp dụng Samba | Ghi "không xác định" |

---

## Kết luận

- Metasploitable 2 có **23 cổng TCP mở** và nhiều dịch vụ cũ, rủi ro cao.
- Nmap là công cụ mạnh để khảo sát bề mặt mạng.
- Hardening (tắt dịch vụ không cần thiết) giúp giảm bề mặt tấn công.
- Chỉ quét hệ thống được phép — bài lab dùng mạng Host-Only cá nhân.

---

