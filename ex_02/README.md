# Bài tập 2 (SS09): Container hóa ứng dụng Spring Boot Microservices

## 1. File `Dockerfile` hoàn chỉnh

Xem trực tiếp file không đuôi mở rộng tại: [`Dockerfile`](./Dockerfile)

```dockerfile
# 1. Chọn base image chứa JRE 17 gọn nhẹ
FROM eclipse-temurin:17-jre-alpine

# 2. Tạo thư mục làm việc trong container
WORKDIR /app

# 3. Copy file jar từ thư mục build của Gradle vào container
COPY build/libs/payment-service-1.0.0.jar app.jar

# 4. Mở port 8080 để giao tiếp với các microservices khác
EXPOSE 8080

# 5. Lệnh khởi chạy ứng dụng Spring Boot
ENTRYPOINT ["java", "-jar", "app.jar"]
```

---

## 2. Giải thích các chỉ thị Docker đã sử dụng

| Chỉ thị | Chức năng |
| :--- | :--- |
| **`FROM`** | Khai báo Base Image làm nền tảng cho container (`eclipse-temurin:17-jre-alpine` chỉ chứa JRE 17 trên nền Alpine Linux nên rất nhẹ, tối ưu cho môi trường chạy thực tế). |
| **`WORKDIR`** | Tạo và thiết lập `/app` làm thư mục làm việc mặc định bên trong container cho mọi lệnh tiếp theo (`COPY`, `ENTRYPOINT`). |
| **`COPY`** | Sao chép tệp thực thi `.jar` đã biên dịch bởi Gradle (`build/libs/payment-service-1.0.0.jar`) từ máy build vào trong `/app/app.jar`. |
| **`EXPOSE`** | Khai báo cổng `8080` mà ứng dụng Spring Boot lắng nghe để các microservices khác hoặc port mapping (`-p`) giao tiếp. |
| **`ENTRYPOINT`** | Định nghĩa lệnh thực thi chính khi container khởi động (`java -jar app.jar`) theo cú pháp exec form giúp tiến trình Java nhận đúng tín hiệu tắt (SIGTERM) từ Docker. |
