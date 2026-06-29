# 📘 Module 09: Docker & Deployment
## ASP.NET Core Ultimate Guide — Zero to Hero

---

# Mục lục
1. [Lịch sử triển khai — Từ thuê nhà đến ở trọ capsule](#1-lịch-sử)
2. [Container = Phòng capsule siêu nhẹ](#2-container)
3. [Docker — Hệ thống quản lý phòng](#3-docker)
4. [Dockerfile — Bản thiết kế phòng](#4-dockerfile)
5. [Docker Compose — Quản lý cả tòa nhà](#5-docker-compose)
6. [Nginx Reverse Proxy — Lễ tân tòa nhà](#6-nginx)
7. [Secrets — Két sắt tòa nhà](#7-secrets)

---

# 1. Lịch sử triển khai — Từ thuê nhà đến ở trọ capsule

| Đời | Mô hình | Ẩn dụ | Vấn đề |
|---|---|---|---|
| 👴 1 | **Bare Metal** | Thuê nguyên CĂN NHÀ cho 1 người ở | Nhà 5 phòng mà ở 1 phòng — lãng phí! |
| 👨 2 | **Virtual Machine** | Chia nhà thành NHIỀU CĂN HỘ (mỗi hộ có bếp, WC riêng) | Mỗi căn hộ phải có WC riêng → tốn diện tích |
| 🧑 3 | **Container** ✅ | Khách sạn CAPSULE — mỗi người 1 ô nhỏ, WC DÙNG CHUNG | Nhẹ, nhanh, tiết kiệm! |

```
Bare Metal:    [App] → [Hệ điều hành] → [Máy vật lý]
VM:            [App] → [HĐH riêng mỗi VM] → [Hypervisor] → [Máy vật lý]
Container:     [App] → [Container] → [1 HĐH dùng chung] → [Máy vật lý]
               ↑ Không cần HĐH riêng → nhẹ hơn nhiều!
```

---

# 2. Container — Phòng capsule siêu nhẹ

Container **KHÔNG phải VM**. Nó dùng tính năng của Linux kernel để cô lập:

### Namespaces — Mỗi capsule có "thế giới riêng"

| Tường cách ly | Ẩn dụ |
|---|---|
| **PID** | Khách trong capsule A không thấy khách capsule B |
| **Network** | Mỗi capsule có WiFi số riêng |
| **Mount** | Mỗi capsule có ngăn tủ riêng |
| **User** | "Chủ phòng" trong capsule ≠ chủ khách sạn |

### CGroups — Giới hạn "tiền điện"
```yaml
# Mỗi capsule chỉ được dùng 512MB RAM, 50% CPU
deploy:
  resources:
    limits:
      cpus: '0.50'
      memory: 512M
```

---

# 3. Docker — Hệ thống quản lý phòng

| Khái niệm | Ẩn dụ |
|---|---|
| **Image** | BẢN THIẾT KẾ phòng capsule (photocopy bao nhiêu phòng cũng được) |
| **Container** | 1 PHÒNG THẬT đang có khách ở (tạo từ bản thiết kế) |
| **Dockerfile** | TỜ CHỈ DẪN xây bản thiết kế |
| **Registry** (Docker Hub) | TRUNG TÂM LƯU TRỮ bản thiết kế (ai cũng tải được) |
| **Volume** | Ổ CỨNG RỜI — dữ liệu không mất khi dọn phòng |
| **Network** | DÂY ĐIỆN THOẠI nội bộ giữa các phòng |

### Lệnh Docker cơ bản

```bash
docker pull mcr.microsoft.com/dotnet/aspnet:8.0  # Tải bản thiết kế
docker run -d -p 8080:80 --name myapp myimage     # Mở phòng mới
docker ps                                          # Xem phòng nào đang mở
docker logs myapp                                  # Đọc nhật ký phòng
docker exec -it myapp bash                         # Ghé thăm phòng đang ở
docker stop myapp && docker rm myapp               # Trả phòng → dọn sạch
```

---

# 4. Dockerfile — Bản thiết kế phòng

## Multi-stage Build — Xưởng + Phòng ở

```dockerfile
# === XƯỞNG SẢN XUẤT (SDK ~700MB — to nhưng có đủ công cụ) ===
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
WORKDIR /src
COPY *.csproj .
RUN dotnet restore         # Mua vật liệu
COPY . .
RUN dotnet publish -c Release -o /app  # Lắp ráp sản phẩm

# === PHÒNG Ở THẬT (Runtime ~100MB — nhẹ, gọn, sạch) ===
FROM mcr.microsoft.com/dotnet/aspnet:8.0
WORKDIR /app
COPY --from=build /app .   # Chỉ lấy sản phẩm, bỏ lại xưởng
EXPOSE 8080
ENTRYPOINT ["dotnet", "MyApp.dll"]  # Mở cửa đón khách!
```

> 💡 **Mẹo**: Xưởng sản xuất (SDK) nặng 700MB nhưng **KHÔNG đi theo** sản phẩm. Phòng ở chỉ 100MB → Image nhẹ, deploy nhanh!

---

# 5. Docker Compose — Quản lý cả tòa nhà

Quán thật cần nhiều phòng: API, Database, Redis, Nginx. Mở từng phòng bằng tay → **mệt chết!**

**Docker Compose** = 1 file YAML mở CẢ TÒA NHÀ bằng 1 lệnh:

```yaml
# docker-compose.yml — BẢN THIẾT KẾ TÒA NHÀ
version: '3.8'

services:
  # Phòng 1: KHO (PostgreSQL)
  postgres:
    image: postgres:16
    environment:
      POSTGRES_USER: admin
      POSTGRES_PASSWORD_FILE: /run/secrets/db_password
      POSTGRES_DB: banking
    volumes:

      - pg-data:/var/lib/postgresql/data  # Ổ cứng rời — dữ liệu không mất
    healthcheck:
      test: ["CMD-SHELL", "pg_isready"]   # Kiểm tra kho đã mở chưa
    secrets:

      - db_password

  # Phòng 2 & 3: BẾP (API — 2 bản sao)
  api:
    build: ./src/MyApp.Api
    environment:

      - ASPNETCORE_URLS=http://+:8080
    depends_on:
      postgres:
        condition: service_healthy  # Chờ kho mở xong mới bật bếp
    deploy:
      replicas: 2                   # Mở 2 bếp song song!
    secrets:

      - db_password

  # Phòng 4: LỄ TÂN (Nginx)
  nginx:
    image: nginx:alpine
    ports:

      - "80:80"  # Cổng duy nhất ra ngoài
    volumes:

      - ./nginx.conf:/etc/nginx/nginx.conf:ro

volumes:
  pg-data:  # Khai báo ổ cứng rời

secrets:
  db_password:
    file: ./secrets/db_password.txt  # Mật khẩu nằm trong file riêng
```

```bash
docker compose up -d           # MỞ CẢ TÒA NHÀ (background)
docker compose ps              # Xem phòng nào đang mở
docker compose logs api        # Đọc nhật ký bếp
docker compose down            # ĐÓNG CẢ TÒA NHÀ
docker compose up -d --scale api=3  # Mở thêm 1 bếp nữa → 3 bếp!
```

---

# 6. Nginx Reverse Proxy — Lễ tân tòa nhà

## 6.1. Tại sao cần Lễ tân?

Khách KHÔNG biết tòa nhà có 3 bếp. Khách chỉ biết **1 cổng vào** (port 80). **Nginx** = Lễ tân đứng cổng, chia khách đều cho 3 bếp:

```
Khách → :80 → Lễ tân (Nginx) → Bếp 1 (:8080)
                              → Bếp 2 (:8080)
                              → Bếp 3 (:8080)
```

## 6.2. Cấu hình Lễ tân

```nginx
http {
    upstream cac_bep {
        server api:8080;  # Docker tự biết "api" là phòng nào
    }

    server {
        listen 80;
        location / {
            proxy_pass http://cac_bep;                # Chuyển khách vào bếp
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;  # Ghi nhớ IP khách thật
        }
    }
}
```

## 6.3. Lợi ích của Lễ tân

| Việc | Ẩn dụ |
|---|---|
| **Load Balancing** | Chia khách đều cho nhiều bếp |
| **SSL Termination** | Lễ tân xử lý ổ khóa HTTPS, bếp không cần |
| **Static Files** | Lễ tân tự phát khăn lạnh (CSS/JS) — nhanh hơn bếp |
| **Security** | Khách KHÔNG biết bếp ở đâu — ẩn hoàn toàn |

---

# 7. Secrets — Két sắt tòa nhà

## 7.1. ĐỪNG BAO GIỜ viết mật khẩu lên bảng

```yaml
# ❌ CHẾT — mật khẩu viết công khai
environment:

  - DB_PASSWORD=MySecretPassword123

# ✅ AN TOÀN — bỏ trong két sắt
secrets:

  - db_password
```

## 7.2. Docker Secrets = Két sắt

Mật khẩu nằm trong file → Docker mount vào `/run/secrets/` (read-only) → chỉ app trong container đọc được.

```csharp
// Đọc từ két sắt
var password = File.ReadAllText("/run/secrets/db_password").Trim();

// Hoặc dùng cấu hình tự động
builder.Configuration.AddKeyPerFile("/run/secrets", optional: true);
```

---

> 🎉 **XIN CHÚC MỪNG!** Bạn đã hoàn thành toàn bộ **ASP.NET Core Ultimate Guide** — từ Zero đến Hero! Giờ hãy bắt tay vào LÀM DỰ ÁN THẬT để biến kiến thức thành kỹ năng! 🚀
