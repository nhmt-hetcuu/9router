# 9Router Docker Stack

Hệ thống AI Gateway định tuyến đa mô hình (LLM Router) kết hợp cùng Headroom proxy để nén prompt và tối ưu chi phí token, tự động cập nhật qua Watchtower và hỗ trợ kết nối từ xa qua Tailscale Funnel / Cloudflare Tunnel.

---

## 🚀 Các dịch vụ trong Stack

| Dịch vụ | Image | Cổng Host | Chức năng |
|---|---|---|---|
| **9router** | `decolua/9router:latest` | `20128:20128` | Giao diện web & API Gateway định tuyến LLM |
| **headroom** | `ghcr.io/chopratejas/headroom:latest` | `8787:8787` | Bộ đệm nén token và prompt caching |
| **watchtower** | `containrrr/watchtower:latest` | *Không mở* | Tự động cập nhật 9router & headroom mỗi 12h |
| **cloudflared** | `cloudflare/cloudflared:latest` | *Không mở* | Tạo Cloudflare Tunnel trỏ về `http://9router:20128` |
| **tailscale** | `tailscale/tailscale:latest` | *Không mở* | Kết nối mạng nội bộ an toàn và chia sẻ qua Tailscale Funnel |

---

## 📁 Cấu trúc thư mục

```text
.
├── .gitignore               # Bỏ qua database và cache runtime
├── docker-compose.yml       # Khai báo toàn bộ các dịch vụ
├── tailscale/
│   └── serve.json           # Cấu hình Tailscale Funnel / HTTPS reverse proxy
└── README.md                # Tài liệu hướng dẫn này
```

---

## 🛠️ Hướng dẫn triển khai

### 1. Yêu cầu hệ thống
- Đã cài đặt Docker và Docker Compose (v2 khuyến nghị).
- Đảm bảo các cổng `20128` và `8787` không bị chiếm bởi ứng dụng khác.

### 2. Cấu hình Tailscale (Tùy chọn)
Nếu sử dụng dịch vụ Tailscale, chỉnh sửa `docker-compose.yml`:
- Thay `TS_AUTHKEY=tskey-auth-xxxxxx` bằng Auth Key được tạo từ trang quản trị [Tailscale Admin Console](https://login.tailscale.com/admin/settings/keys).

### 3. Khởi chạy
```bash
docker compose up -d
```

### 4. Truy cập dịch vụ
- **Web UI & API:** `http://<IP-Server>:20128`
- **Headroom Proxy:** `http://<IP-Server>:8787`

---

## 🔄 Tự động cập nhật với Watchtower
Container Watchtower được gắn nhãn độc lập:
```yaml
labels:
  - com.centurylinklabs.watchtower.scope=9router
```
Chỉ quét và cập nhật các container thuộc scope `9router`, chạy định kỳ mỗi 12 giờ (`--interval 43200`) và tự động dọn dẹp các image cũ (`--cleanup`).

---

## 📋 Lệnh quản trị thường dùng

```bash
# Xem trạng thái các container
docker compose ps

# Xem nhật ký theo thời gian thực
docker compose logs -f 9router
docker compose logs -f headroom

# Khởi động lại dịch vụ
docker compose restart

# Dừng stack
docker compose down
```
