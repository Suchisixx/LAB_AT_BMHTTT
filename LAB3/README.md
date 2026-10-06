# LAB 5 – THIẾT LẬP MÔ HÌNH TƯỜNG LỬA PFSENSE

## 📌 Giới thiệu
Bài thực hành này xây dựng mô hình mạng doanh nghiệp thu nhỏ với tường lửa **pfSense CE** bảo vệ vùng **LAN** và **DMZ**. Sinh viên thực hiện cấu hình WAN, LAN, DMZ, NAT, và các rule firewall theo tình huống thực tế.

---

## 👨‍🎓 THÔNG TIN SINH VIÊN
| Thông tin | Chi tiết |
| :--- | :--- |
| **Họ và tên** | Vũ Bá Lực |
| **Mã số sinh viên (MSSV)** | 1150080104 |
| **Lớp** | 11_ĐH_THMT |
| **Học phần** | An toàn và Bảo mật Thông tin |
| **Tên bài thực hành** | LAB 5 – Thiết lập mô hình tường lửa pfSense |

---

## 🖥️ Môi trường & Công cụ
| Thành phần | Thông tin |
| :--- | :--- |
| **Máy thật (Host)** | Windows 11 |
| **Phần mềm ảo hóa** | Oracle VirtualBox & VMware Workstation |
| **pfSense** | pfSense CE 2.7.2-RELEASE (amd64) |
| **Windows Server** | Windows Server 2025 |
| **Card mạng** | WAN: Bridged, LAN: Host-Only (`10.0.0.0/8`), DMZ: Internal Network (`172.16.0.0/16`) |

---

## 🗺️ Mô hình mạng

- **LAN:** `10.0.0.0/8` – Domain Controller (`10.0.0.2`), Máy thật (`10.0.0.100`)
- **DMZ:** `172.16.0.0/16` – Web Server (`172.16.0.2`)
- **WAN:** DHCP/NAT – Kết nối Internet qua Bridged Adapter

---

---

## 🚀 Hướng dẫn cài đặt & cấu hình

### 1. Tạo máy ảo pfSense
- **Name:** `pfSense`
- **Type:** BSD, **Version:** FreeBSD (64-bit)
- **RAM:** 2GB, **CPU:** 2 vCPU, **Ổ cứng:** 16GB
- **Card mạng:**
  - Adapter 1 (WAN): Bridged Adapter
  - Adapter 2 (LAN): Host-Only Adapter
  - Adapter 3 (DMZ): Internal Network, Name = `dmz-net`

### 2. Cài đặt pfSense
- Gắn ISO `pfSense-CE-2.7.2-RELEASE-amd64.iso`
- Chọn `Install` → `Auto (UFS)` → `OK`
- Sau khi cài xong, tháo ISO và khởi động lại

### 3. Cấu hình LAN trên console pfSense
- Chọn `Set interface(s) IP address` → chọn LAN
- Đặt IP: `10.0.0.1/8`, tắt DHCP

### 4. Cấu hình máy thật (Windows 11)
- Card Host-Only: IP `10.0.0.100/8`, Subnet `255.0.0.0`
- **Không** đặt Gateway/DNS

### 5. Cấu hình Windows Server (Domain Controller)
- Card LAN: IP `10.0.0.2/8`, Gateway `10.0.0.1`, DNS `10.0.0.2`
- Cài AD DS, promote thành Domain Controller `vietnam.local`
- Cấu hình DNS Forwarder `8.8.8.8`

### 6. Cấu hình DMZ
- Gán card mạng thứ 3 thành OPT1
- Đặt IP `172.16.0.1/16`, Enable interface
- Tạo rule Pass DMZ → Any

### 7. Kiểm tra Outbound NAT
- Vào **Firewall** → **NAT** → **Outbound**
- Chọn **Hybrid Outbound NAT rule generation**
- Kiểm tra Automatic Rules cho LAN và DMZ

### 8. Chuẩn hóa ruleset LAN
- Disable 2 rule Default allow LAN to any (IPv4/IPv6)
- Giữ Anti-Lockout Rule
- Tạo rule nền tảng: Pass LAN net → Any
- Reset States

---

## 🧪 Các tình huống kiểm thử

| STT | Tình huống | Mục tiêu | Kết quả |
| :---: | :--- | :--- | :---: |
| **TH1** | Cấu hình nền tảng | Cấu hình 3 card mạng, LAN IP, DMZ, NAT, rule nền tảng | PASS |
| **TH2** | Kiểm tra rule nền tảng | Test ping/DNS/HTTPS từ Domain Controller | PASS |
| **TH3** | Chặn ICMP nhưng cho phép Web/DNS | Tạo 3 rule: Block ICMP, Pass DNS, Pass HTTP/HTTPS | PASS |
| **TH4** | Chỉ cho một host cụ thể ra Internet | Pass `10.0.0.100`, Block LAN net | PASS |
| **TH5** | Cô lập DMZ khỏi LAN | Block DMZ → LAN, Pass DMZ → Any | FAIL |
| **TH6** | Port Forward WAN → DMZ | NAT Port Forward WAN:8080 → DMZ-Web:80 | PASS |
| **TH7** | Bật logging & đọc Firewall Log | Bật log cho rule Block, đọc log tại Status → System Logs | PASS |
| **TH8** | Cleanup & Khôi phục | Disable rule tình huống, Enable rule nền tảng, Reset States | PASS |

---

## 🔍 Troubleshooting

| Vấn đề | Nguyên nhân | Cách khắc phục |
| :--- | :--- | :--- |
| Không ping được pfSense từ máy thật | Card Host-Only chưa đặt IP `10.0.0.100/8` | Đặt IP cho card Host-Only |
| Ping `8.8.8.8` bị `Destination host unreachable` | Chưa có Default Gateway `10.0.0.1` | Đặt Gateway cho Windows Server |
| Không truy cập được WebGUI pfSense | Card Host-Only chưa cùng dải với pfSense | Đặt IP `10.0.0.100/8`, ping `10.0.0.1` |
| Rule Block ICMP chặn luôn ping vào pfSense | Rule Block nằm trên Anti-Lockout | Đảm bảo Anti-Lockout Enabled và nằm trên cùng |
| pfSense khởi động lâu | Máy thật yếu, chạy nhiều máy ảo | Tắt bớt máy ảo, tăng RAM |
| Windows Server không tạo được Domain Controller | Tài khoản Administrator chưa có mật khẩu | Đặt mật khẩu bằng `net user Administrator P@ssw0rd123` |
| Thứ tự rule sai khiến rule Block bị vô hiệu | Rule Block nằm dưới rule Pass Any | Kéo thả rule Block lên trên cùng |

---

## 📝 Ghi chú
- Báo cáo được xây dựng dựa trên mẫu LAB 3, áp dụng cho bài LAB 5 – Thiết lập mô hình tường lửa pfSense.
- Các tình huống từ TH3 đến TH7 được thực hiện dựa trên hướng dẫn trong tài liệu Lab 5.
- Do hạn chế về tài nguyên máy thật, một số tình huống được kiểm thử ở mức cơ bản và ghi nhận kết quả theo đúng quy trình.

---

