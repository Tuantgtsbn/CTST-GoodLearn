# GoodLearn - Hệ sinh thái Học Tập AI

## Clone

```bash
git clone --recurse-submodules https://github.com/Tuantgtsbn/CTST-GoodLearn.git

```

## Tổng quan

GoodLearn là một dự án nền tảng học tập thông minh kết hợp nhiều thành phần:

- `BE_GoodLearn-master`: backend chính của hệ thống học tập AI.
- `FE_GoodLearn-master`: frontend ứng dụng React/Tailwind hiển thị giao diện người dùng.
- `GoodLearn-ServiceTTS-main`: dịch vụ Text-to-Speech (TTS) tiếng Việt/Anh.
- `GoodLearn-SongService-master`: dịch vụ chấm điểm giọng hát và phân tích vocal.

Dự án hướng tới xây dựng một nền tảng học tập trực tuyến cho học sinh và người học suốt đời, tích hợp AI cho bài kiểm tra, hỏi đáp, video học, flashcard và các dịch vụ âm thanh.

---

## Cấu trúc thư mục

```text
GoodLearnEnd/
├── BE_GoodLearn-master/           # Backend chính (Express + Prisma)
├── FE_GoodLearn-master/           # Frontend chính (React + Vite)
├── GoodLearn-ServiceTTS-main/     # Service TTS tiếng Việt/Anh
├── GoodLearn-SongService-master/  # Service chấm điểm giọng hát
└── README.md                      # Tài liệu tổng quan dự án
```

### Backend chính (`BE_GoodLearn-master`)

- `src/app.ts`, `src/server.ts`: khởi tạo Express app và entry point server.
- `src/config/`: cấu hình Prisma, Swagger, MinIO, JWT, môi trường.
- `src/controllers/`: route handlers.
- `src/services/`: logic nghiệp vụ.
- `src/middleware/`: xử lý auth, lỗi, validation.
- `src/routes/`: khai báo API.
- `src/dtos/`, `src/types/`: kiểu dữ liệu và DTO.
- `prisma/schema.prisma`: mô hình dữ liệu PostgreSQL.

### Frontend chính (`FE_GoodLearn-master`)

- `src/App.tsx`, `src/main.tsx`: root React app.
- `src/components/`: UI component và layout.
- `src/pages/`: các trang ứng dụng.
- `src/routes/`: cấu hình route và bảo mật.
- `src/api/`: client Axios, query key, fetcher.
- `src/redux/`: store và slices.
- `src/styles/`: Tailwind/SCSS global styles.

### Dịch vụ TTS (`GoodLearn-ServiceTTS-main`)

- Dự án TTS dựa trên **VieNeu-TTS**, hỗ trợ tiếng Việt và tiếng Anh.
- Hỗ trợ **voice cloning**, **Turbo mode** cho CPU, **GPU server** và chạy bằng `uv`.
- Có file `README.md` riêng để hướng dẫn chi tiết cài đặt và chạy.

### Dịch vụ Song/Scoring (`GoodLearn-SongService-master`)

- Backend FastAPI chấm điểm giọng hát.
- Hỗ trợ phân tích pitch, rhythm, stability, dynamics.
- Có Docker và cấu hình `uvicorn`.

---

## Hướng dẫn cài đặt nhanh

### 1. Backend chính

```bash
cd BE_GoodLearn-master
npm install
```

Sử dụng chế độ phát triển:

```bash
npm run dev
```

Xây dựng và khởi động production:

```bash
npm run build
npm run start
```

Các lệnh hỗ trợ khác:

- `npm run lint`
- `npm run lint:fix`
- `npm run test`
- `npm run worker:email`
- `npm run worker:video`
- `npm run worker:flashcard`

> Lưu ý: dự án backend sử dụng Node.js 18+, Prisma, Redis, MinIO và Swagger.

### 2. Frontend chính

```bash
cd FE_GoodLearn-master
npm install
npm run dev
```

Hoặc xây dựng production:

```bash
npm run build
npm run preview
```

> Frontend sử dụng React 19, Vite, Tailwind CSS, Redux Toolkit, TanStack Query và MUI.

### 3. Dịch vụ TTS

```bash
cd GoodLearn-ServiceTTS-main
```

Trong thư mục `GoodLearn-ServiceTTS-main`, có thể cài đặt theo hướng dẫn nội bộ:

- `uv sync` hoặc `pip install -r requirements_xpu.txt`
- chạy giao diện web TTS bằng `uv run vieneu-web`

Hoặc dùng Docker nếu cần môi trường cách ly.

> Xem thêm chi tiết cài đặt và lệnh khởi động trong `GoodLearn-ServiceTTS-main/README.md`.

### 4. Dịch vụ Song/Scoring

```bash
cd GoodLearn-SongService-master
python -m venv venv
# Windows PowerShell
venv\Scripts\Activate.ps1
# hoặc CMD
venv\Scripts\activate.bat
pip install -r requirements.txt
uvicorn main:app --reload --port 8000
```

Hoặc chạy bằng Docker Compose:

```bash
docker-compose up --build
```

> Server sẽ chạy ở `http://localhost:8000` và có thể mở Swagger UI nếu được cấu hình.

---

## Các công nghệ chính

### Backend

- Node.js 18+
- Express.js
- PostgreSQL + Prisma
- Redis
- MinIO
- BullMQ
- JWT
- Swagger/OpenAPI
- Jest
- TypeScript
- Zod/Joi

### Frontend

- React + Vite
- Tailwind CSS
- Redux Toolkit
- TanStack Query
- Axios
- Material UI (MUI)
- TypeScript

### Dịch vụ bổ trợ

- Python + FastAPI (SongService)
- VieNeu-TTS / Python TTS (ServiceTTS)
- Docker / Docker Compose

---

## Ghi chú

- Mỗi thư mục con là một ứng dụng độc lập với hướng dẫn riêng.
- Đảm bảo cài đặt đúng phiên bản Node, Python và Docker khi sử dụng các dịch vụ.
- Xem README riêng trong `GoodLearn-ServiceTTS-main` và `GoodLearn-SongService-master` để biết nội dung setup đầy đủ.
