# BÁO CÁO THỰC HÀNH LAB 6
## Xác thực Active Directory và Chính sách truy cập trên pfSense

---

## 1. Thông tin sinh viên
- **Họ và tên:** Vũ Bá Lực
- **MSSV:** 1150080104
- **Lớp:** 11_ĐH_THMT
- **Học phần:** An toàn và Bảo mật Thông tin

---

## 2. Môi trường & Công cụ

| Thành phần | Phiên bản |
|------------|-----------|
| VirtualBox | 7.0.x |
| pfSense CE | 2.7.2-RELEASE |
| Windows Server | 2022 Standard |
| Ubuntu Server | 26.04.1 LTS |
| Kali Linux | Rolling |

## 4. Kết quả kiểm thử

| STT | Tình huống | Kết quả |
|:---:|:-----------|:-------:|
| TH1 | Xác thực LDAP | PASS |
| TH2 | Captive Portal | PASS |
| TH3 | KB1 - JOB_SITE | PASS |
| TH4 | KB2 - Restricted | PASS |
| TH5 | KB3 - Social Schedule | PASS |
| TH6 | KB4 - Work Hours | KHÔNG THỰC HIỆN |
| TH7 | KB5 - DMZ Isolation | KHÔNG THỰC HIỆN |

---

## 5. Vấn đề và cách khác phục

> **Lỗi 1:** LDAP Authentication failed
>
> **Khắc phục:** Kiểm tra Bind DN, password, ping 10.6.0.10 thử.

> **Lỗi 2:** pfSense lỗi PHP 
>
> **Khắc phục:** Vào Single User Mode, copy backup từ `/cf/conf/backup/`.

> **Lỗi 3:** pfSense lỗi cấu hình (config.xml) `
>
> **Khắc phục:** Trước khi power off máy ảo phải tắt cẩn thận bằng menu trong pfSense với mục 6, không được tắt đột ngột, để backup file xml coi bước trên `/cf/conf/backup/`.

> **Lỗi 4:** Gửi yêu cầu gói tin bị lỗi trong các kịch bản
>
> **Khắc phục:** Để ý thứ tự các Rule trong pfSense, đảm bảo đúng thứ tự trình bày, còn nữa là nhớ tắt các Rule không cần thiết để thực hành.

---

## 6. Checklist nộp bài

- [x] Topology VirtualBox 4 adapter
- [x] LDAP test sv_a/sv_b
- [x] Captive Portal login
- [x] Alias & Schedule
- [x] 5 kịch bản rule + test
- [x] Firewall Log & State Table
- [x] Bảng kết quả & 12 câu hỏi
- [x] File config.xml backup
