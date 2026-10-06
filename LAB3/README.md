# LAB 5 – THIẾT LẬP MÔ HÌNH TƯỜNG LỬA PFSENSE

## 📌 Giới thiệu
Bài thực hành này xây dựng mô hình mạng doanh nghiệp thu nhỏ với tường lửa **pfSense CE** bảo vệ vùng **LAN** và **DMZ**. Sinh viên thực hiện cấu hình WAN, LAN, DMZ, NAT, và các rule firewall theo tình huống thực tế.

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

![Mô hình Lab 5](https://github.com/your-username/your-repo/raw/main/images/topology.png)

- **LAN:** `10.0.0.0/8` – Domain Controller (`10.0.0.2`), Máy thật (`10.0.0.100`)
- **DMZ:** `172.16.0.0/16` – Web Server (`172.16.0.2`)
- **WAN:** DHCP/NAT – Kết nối Internet qua Bridged Adapter

---

## 📂 Cấu trúc thư mục
