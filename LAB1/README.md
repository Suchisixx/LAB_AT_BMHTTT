# Lab 1: Bắt gói tin Telnet - SSH

**Họ và tên** : Vũ Bá Lực 
**Mã số sinh viên** : 1150080104 
**Lớp** : 11-ĐH-THMT 
**Môn học** :  An toàn Bảo mật Hệ thống Thông tin 

## Tên bài Lab

**Lab 1: Examining SSH & Telnet in Wireshark**  
(Bắt gói tin Telnet - SSH)

## Mục tiêu

- Giả lập mạng Telnet – SSH Client/Server.
- Tìm hiểu cơ chế bắt gói tin bằng Wireshark.
- So sánh sự khác biệt về bảo mật giữa Telnet và SSH.

## Nội dung đã thực hiện

### Phần 1: Cài đặt môi trường

#### Trên Kali Linux

```bash
# Cài Telnet Server + Client + SSH Client
sudo apt update
sudo apt install -y xinetd telnetd inetutils-telnet openssh-client

# Tạo file cấu hình xinetd cho Telnet
sudo tee /etc/xinetd.d/telnet > /dev/null <<'EOF'
service telnet
{
    flags           = REUSE
    socket_type     = stream
    wait            = no
    user            = root
    server          = /usr/sbin/telnetd
    log_on_failure  += USERID
    disable         = no
}
EOF

# Khởi động xinetd
sudo systemctl restart xinetd

# Kiểm tra port 23
ss -ltn | grep ':23'

# Tạo tài khoản 
sudo adduser lucvu-lab1
# Password: 1150080104
```

#### Trên Window 11
```bash
# Cài OpenSSH Server
Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0

# Khởi động
Start-Service sshd
Set-Service -Name sshd -StartupType Automatic

# Mở firewall
New-NetFirewallRule -Name sshd -DisplayName 'OpenSSH Server (sshd)' -Enabled True -Direction Inbound -Protocol TCP -Action Allow -LocalPort 22

# Tạo tài khoản 
net user uitlab lucvu-lab1 /add
```
### Phần 2: Bắt gói tin
#### 2.1 Telnet 
```
Mở Wireshark trên Kali và chọn interface Loopback: lo
Capture Filter: tcp.port == 23

Mở Terminal khác, chạy:
telnet 127.0.0.1
Đăng nhập: lucvu-lab1 / 1150080104

Dừng Wireshark, tìm gói TELNET → Follow → TCP Stream
Lưu file: telnet_capture.pcapng
```

#### 2.2 SSH
```
Mở Wireshark trên Kali chọn interface eth0
Capture Filter: tcp.port == 22

Mở Terminal khác, chạy:
ssh uitlab@192.168.1.41
Gõ yes khi yêu cầu để chấp nhận host key

Nhập password: 1150080104

Gõ vài lệnh:

Dừng Wireshark → chuột phải gói SSH → Follow → TCP Stream
Lưu file: ssh_capture.pcapng
```

## Kết quả thực hiện
Bắt được gói tin telnet và SSH, hiểu được sự khác biệt giữa cả 2 

## Các lưu ý cần thiết
Sau bài lab em thấy cần kiểm tra kĩ IP để tốn thời gian chạy lại lệnh, setup và tải các gói trước để tránh mất thời gian, đôi khi có những lệnh cú pháp khác so với OS đó nên cần tìm hiểu thêm
