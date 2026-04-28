# Hướng Dẫn Deploy GoodLearn Platform lên VPS (Ubuntu)

Dưới đây là các bước chi tiết để đưa toàn bộ hệ thống GoodLearn lên môi trường Production (VPS) với NGINX làm Reverse Proxy.

## 1. Yêu cầu hệ thống VPS
- **OS**: Ubuntu 20.04 hoặc 22.04.
- **Phần mềm cần cài đặt**:
  - Docker & Docker Compose (cho DB, Redis, MinIO)
  - Node.js (v18+) & npm
  - PM2 (quản lý process Node.js & Python) `npm install -g pm2`
  - Python 3.10+ (cho `vocal-scoring-backend` và `VieNeu`)
  - Nginx

## 2. Chuẩn bị source code
Clone repo code về VPS tại thư mục `/var/www/goodlearn` (hoặc thư mục bạn chọn).

```bash
cd /var/www/goodlearn/Sourcecode
```

## 3. Khởi chạy Docker Compose (Services hạ tầng)
Hệ thống sử dụng Docker cho PostgreSQL, Redis và MinIO.

```bash
cd Backend
# Chạy các service ở background
docker-compose up -d
```

## 4. Cấu hình Biến Môi Trường (Environment)
Đảm bảo đã tạo và điều chỉnh file `.env.production` tại các vị trí:
- `Backend/.env.production` (Như tôi đã setup giúp bạn, nhớ điền đúng mật khẩu thật, config mail và API key vào)
- `Frontend/.env.production` (Đã được cấu hình với domain `https://goodlearn.id.vn`)

## 5. Khởi chạy Backend (PM2 Ecosystem)
Backend bao gồm API server và 3 workers (email, video, flashcard).

```bash
cd /var/www/goodlearn/Sourcecode/Backend
# Cài đặt dependency
npm install
# Khởi chạy Prisma migrate và generate client
npx prisma generate
npx prisma migrate deploy
# Build source TypeScript
npm run build
# Start qua pm2 (ecosystem.config.js)
npm run start:pm2
```

## 6. Build và Serve Frontend
Frontend sẽ build static file và chạy preview (port 3000) thông qua PM2.

```bash
cd /var/www/goodlearn/Sourcecode/Frontend
npm install
npm run build
# Sử dụng pm2 để chạy server preview ở port 3000
pm2 start "npm run preview" --name "goodlearn-frontend"
```

## 7. Khởi chạy Vocal Scoring Backend
Backend xử lý chấm điểm giọng hát bằng Python.

```bash
cd /var/www/goodlearn/Sourcecode/vocal-scoring-backend
# Tạo virtual environment
python3 -m venv venv
source venv/bin/activate
# Cài thư viện
pip install -r requirements.txt
# Chạy với PM2 bằng uvicorn (chạy ở port 8000)
pm2 start "uvicorn main:app --host 127.0.0.1 --port 8000" --name "goodlearn-vocal"
```

## 8. Khởi chạy VieNeu service
Bạn cần clone source từ repository Github của VieNeu.

```bash
cd /var/www/goodlearn/Sourcecode
git clone https://github.com/<tên_tài_khoản>/VieNeu.git
cd VieNeu
# Khởi chạy theo hướng dẫn của repo VieNeu, ví dụ:
# pip install -r requirements.txt
# pm2 start "python main.py" --name "goodlearn-vieneu"
```

## 9. Cấu Hình NGINX
Tôi đã tạo sẵn file `nginx.conf` cho bạn ở thư mục `Sourcecode`. Bạn copy nội dung file này vào cấu hình Nginx của VPS.

```bash
# Copy file cấu hình
sudo cp /var/www/goodlearn/Sourcecode/nginx.conf /etc/nginx/sites-available/goodlearn.id.vn
# Link sang sites-enabled
sudo ln -s /etc/nginx/sites-available/goodlearn.id.vn /etc/nginx/sites-enabled/
# Xoá file default (nếu cần)
sudo rm /etc/nginx/sites-enabled/default
# Test cấu hình
sudo nginx -t
# Restart Nginx
sudo systemctl restart nginx
```

## 10. Đăng ký SSL (HTTPS) cho domain
Sử dụng Certbot để tự động lấy chứng chỉ Let's Encrypt cho `goodlearn.id.vn`:

```bash
sudo apt install certbot python3-certbot-nginx
sudo certbot --nginx -d goodlearn.id.vn
```
(Certbot sẽ tự động sửa file cấu hình Nginx để thêm SSL)

## 11. Lưu cấu hình PM2
Để PM2 tự động khởi động lại tất cả app (Frontend, Backend, Workers, AI) mỗi khi VPS khởi động lại:

```bash
pm2 save
pm2 startup
```

Chúc mừng bạn đã deploy thành công! Truy cập trang web: `https://goodlearn.id.vn` để trải nghiệm.
