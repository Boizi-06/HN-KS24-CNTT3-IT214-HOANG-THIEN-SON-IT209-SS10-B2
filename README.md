# Bài 2 – Đóng gói Backend bằng Single-stage Dockerfile

## Mục tiêu

Đóng gói một REST API đơn giản dùng Node.js và Express bằng Dockerfile single-stage, sau đó chạy container và kiểm tra API qua port mapping.

## Công nghệ sử dụng

- Node.js 18 Alpine
- Express 4
- Docker

## Cấu trúc thư mục

```text
Bai_2/
├── Dockerfile
├── .dockerignore
├── package.json
├── server.js
└── README.md
```

## API

`GET /` trả HTTP `200 OK` cùng JSON:

```json
{
  "status": "success",
  "message": "Hello from Docker Single-stage!"
}
```

Ứng dụng đọc cổng từ `process.env.PORT` và mặc định dùng `3000`. Server lắng nghe trên `0.0.0.0` để có thể truy cập từ bên ngoài container.

## Các chỉ thị trong Dockerfile

- `FROM node:18-alpine`: chọn Node.js 18 trên Alpine Linux làm base image.
- `WORKDIR /app`: đặt thư mục làm việc trong image.
- `ENV PORT=3000`: cấu hình cổng mặc định cho ứng dụng.
- `COPY package*.json ./`: chép manifest trước để cài dependency.
- `RUN npm install --omit=dev`: cài dependency cần thiết khi chạy ứng dụng.
- `COPY . .`: chép mã nguồn vào image.
- `EXPOSE 3000`: khai báo cổng ứng dụng trong container.
- `CMD ["node", "server.js"]`: khởi chạy ứng dụng bằng exec form.

## Build image

Chạy từ thư mục `Bai_2`:

```bash
docker build -t hw-backend:v1 .
```

## Chạy container

```bash
docker run -d --name app-single-stage -p 8080:3000 hw-backend:v1
```

Port mapping `8080:3000` ánh xạ cổng `8080` trên máy host đến cổng `3000` trong container. Vì vậy, client gọi `http://localhost:8080/`, còn ứng dụng bên trong container lắng nghe ở cổng `3000`.

## Xem trạng thái và log

```bash
docker ps
docker logs app-single-stage
```

Log khởi động dự kiến có dòng `Server listening on port 3000`.

## Kiểm tra API

```bash
curl -i http://localhost:8080/
```

Kiểm tra riêng HTTP status:

```bash
curl -s -o /dev/null -w "%{http_code}\n" http://localhost:8080/
```

Kết quả mong đợi là HTTP `200`, Content-Type JSON và body có `"status":"success"` cùng `"message":"Hello from Docker Single-stage!"`.
