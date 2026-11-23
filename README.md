# Homework Session 09 - GitLab CI/CD Pipeline & Dockerize Spring Boot

Kho lưu trữ toàn bộ bài tập thực hành **Session 09 - DevOps** (GitLab CI/CD Pipeline cho ứng dụng Java Spring Boot Microservices).

---

## Danh Mục Bài Tập

| Bài tập | Nội dung chính | Thư mục | Tệp nộp chính | Trạng thái |
| :--- | :--- | :--- | :--- | :---: |
| **Bài tập 1** | Khắc phục sự cố Pipeline CI/CD cơ bản (lỗi `stages` YAML & thiếu `image` JDK 17) | [`ex_01/`](ex_01/) | `.gitlab-ci.yml`, `README.md` | Hoàn thành |
| **Bài tập 2** | Container hóa Spring Boot Microservices (`payment-service-1.0.0.jar`) | [`ex_02/`](ex_02/) | `Dockerfile`, `README.md` | Hoàn thành |
| **Bài tập 3** | Pipeline 2 stage `test` + `build` kết nối PostgreSQL (`services`) & lưu `artifacts` | [`ex_03/`](ex_03/) | `.gitlab-ci.yml`, `README.md` | Hoàn thành |
| **Bài tập 4** | Bổ sung stage `dockerize` (DinD) build & push image lên Docker Hub với CI/CD Variables | [`ex_04/`](ex_04/) | `.gitlab-ci.yml`, `Dockerfile`, `README.md` | Hoàn thành |

---

## Tóm tắt kỹ thuật từng bài

1. **Bài tập 1 ([ex_01](ex_01/)):** Sửa `stages` thành dạng mảng YAML `- build_app`, thêm `image: gradle:7.6-jdk17` và `chmod +x gradlew`.
2. **Bài tập 2 ([ex_02](ex_02/)):** Hoàn thiện `Dockerfile` chuẩn với `FROM eclipse-temurin:17-jre-alpine`, `WORKDIR /app`, `COPY`, `EXPOSE 8080`, `ENTRYPOINT ["java", "-jar", "app.jar"]`.
3. **Bài tập 3 ([ex_03](ex_03/)):** Thiết lập `services: postgres:14-alpine` cho Integration Test, `artifacts: paths: build/libs/*.jar` với `expire_in: 1 day`.
4. **Bài tập 4 ([ex_04](ex_04/)):** Cấu hình stage `dockerize` sử dụng `docker:dind`, `dependencies: [build_job]` để nhận JAR từ stage trước, đăng nhập bảo mật qua `$DOCKER_USERNAME` / `$DOCKER_PASSWORD` (`--password-stdin`) và đẩy image lên Docker Hub.
