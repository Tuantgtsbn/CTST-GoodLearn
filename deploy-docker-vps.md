# Hướng Dẫn Deploy Toàn Bộ Hệ Thống GoodLearn Bằng Docker (VPS Ubuntu)

Với cách tiếp cận sử dụng **Docker** và **Docker Compose**, mọi thứ đã được container hóa hoàn toàn. Phương pháp này giúp bạn giảm thiểu tối đa các lỗi môi trường, dễ dàng quản lý và tự khởi động lại khi sập.

## 1. Yêu Cầu VPS & Cài Đặt Ban Đầu

- **Hệ điều hành**: Ubuntu 20.04 hoặc 22.04 LTS
- **RAM**: Khuyến nghị từ 4GB trở lên (Do AI model và nhiều container chạy cùng lúc).

Đăng nhập vào VPS và chạy lệnh cài đặt **Docker** & **Nginx**:

```bash
# Cài đặt Nginx
sudo apt update
sudo apt install -y nginx certbot python3-certbot-nginx

# Cài đặt Docker
sudo apt install -y apt-transport-https ca-certificates curl software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin
```

## 2. Chuẩn Bị Source Code

Đưa toàn bộ thư mục `Sourcecode` của bạn lên VPS. Thông thường, bạn đưa nó vào thư mục `/var/www/goodlearn`.

```bash
mkdir -p /var/www/goodlearn
cd /var/www/goodlearn

# Clone code của bạn về đây, đảm bảo bạn đang ở folder chứa file docker-compose.yml gốc
```

### Xử lý service VieNeu (Bắt buộc)

Vì `VieNeu` là một repository từ Github nên bạn cần clone trực tiếp nó về VPS, ngang hàng với `Backend` và `Frontend`.

```bash
git clone https://github.com/tên_repo_của_bạn/VieNeu.git
```

_(Lưu ý: Bạn tự config riêng Docker cho VieNeu theo doc của repository đó, hoặc khởi chạy bằng PM2 như hướng dẫn trong docs)._

## 3. Cấu Hình Biến Môi Trường (.env)

Tôi đã cập nhật sẵn file `Backend/.env.production`. Tuy nhiên, trước khi start, bạn cần mở file này ra và điền các API key thực tế:

- `EMAIL_PASS`, `RESEND_API_KEY`
- `FAL_USER_API_KEY`, `OPENROUTER_API_KEY`
- Thay đổi `SESSION_SECRET_KEY`, ... nếu muốn.

_(Chú ý: Trong file `Backend/.env.production`, các đường dẫn kết nối đã được tôi chỉnh thành tên container của Docker như `postgres`, `redis`, `minio`, `vocal-scoring`. Hãy **giữ nguyên** cấu trúc này để các container nhận ra nhau)_.

## 4. Khởi Chạy Toàn Bộ Hệ Thống Bằng Docker

Tại thư mục chứa file `docker-compose.yml` gốc (thư mục `Sourcecode`), chạy lệnh sau để build và khởi động hệ thống:

```bash
sudo docker compose up -d --build
```

Lệnh này sẽ:

1. Kéo image `postgres`, `redis`, `minio`.
2. Tự động Build Dockerfile cho `vocal-scoring-backend` (Cài python, AI ffmpeg).
3. Tự động Build Dockerfile cho `Backend` (Cài TS, pm2-runtime, Prisma).
4. Tự động Build Dockerfile cho `Frontend` (Cài Vite, build static).
5. Khởi chạy toàn bộ và tự động mở các port `3000`, `5000`, `8000`, `5433`, `9000`, `9001` ra môi trường VPS.

Kiểm tra trạng thái các container:

```bash
sudo docker compose ps
```

## 5. Cấu Hình Reverse Proxy Nginx & SSL

Bây giờ, Nginx sẽ đứng mũi chịu sào để nhận traffic từ tên miền `goodlearn.id.vn` và phân phát vào các port của Docker.

```bash
# Xóa cấu hình mặc định (nếu có)
sudo rm /etc/nginx/sites-enabled/default

# Copy cấu hình nginx.conf vào sites-available
sudo cp /var/www/goodlearn/Sourcecode/nginx.conf /etc/nginx/sites-available/goodlearn.id.vn

# Tạo symlink để kích hoạt
sudo ln -s /etc/nginx/sites-available/goodlearn.id.vn /etc/nginx/sites-enabled/

# Kiểm tra lỗi cấu hình
sudo nginx -t

# Khởi động lại Nginx
sudo systemctl restart nginx
```

## 6. Đăng Ký Chứng Chỉ SSL (HTTPS)

Để web có thể chạy mượt mà không bị lỗi bảo mật trên trình duyệt, tiến hành lấy HTTPS cho Nginx qua Certbot:

```bash
sudo certbot --nginx -d goodlearn.id.vn
```

Certbot sẽ yêu cầu nhập email và sẽ tự cấu hình lại file nginx cho bạn.

---

### Một Số Lệnh Quản Trị Hữu Ích:

- Xem log của toàn hệ thống: `sudo docker compose logs -f`
- Xem log riêng của Backend: `sudo docker compose logs -f backend`
- Dừng toàn bộ hệ thống: `sudo docker compose down`
- Cập nhật lại code và chạy lại: `sudo docker compose up -d --build`

Chúc mừng bạn! Cấu trúc Docker này được thiết kế theo chuẩn microservices hiện đại, tính ổn định rất cao cho Production.

```
docker exec -it goodlearn-backend npx tsx src/config/minioClient.ts
```
