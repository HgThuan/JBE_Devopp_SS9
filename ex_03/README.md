# Bài tập 3 (SS09): Xây dựng Pipeline CI/CD kết nối PostgreSQL

## 1. File `.gitlab-ci.yml` hoàn chỉnh

Xem trực tiếp tại: [`.gitlab-ci.yml`](./.gitlab-ci.yml)

```yaml
image: gradle:7.6-jdk17

stages:
  - test
  - build

cache:
  key: "$CI_COMMIT_REF_SLUG"
  paths:
    - .gradle/wrapper
    - .gradle/caches

test_job:
  stage: test
  services:
    - name: postgres:14-alpine
      alias: postgres
  variables:
    POSTGRES_DB: hrms_test_db
    POSTGRES_USER: postgres
    POSTGRES_PASSWORD: test_password
    POSTGRES_HOST_AUTH_METHOD: trust
    SPRING_DATASOURCE_URL: jdbc:postgresql://postgres:5432/hrms_test_db
    SPRING_DATASOURCE_USERNAME: postgres
    SPRING_DATASOURCE_PASSWORD: test_password
  before_script:
    - chmod +x gradlew
  script:
    - ./gradlew test --info

build_job:
  stage: build
  before_script:
    - chmod +x gradlew
  script:
    - ./gradlew build -x test
  artifacts:
    name: "app-jar-$CI_COMMIT_SHORT_SHA"
    paths:
      - build/libs/*.jar
    expire_in: 1 day
```

---

## 2. Giải thích cơ chế kỹ thuật

1. **Chạy tuần tự nhiều stages (`stages: [test, build]`):**
   - GitLab CI đảm bảo `build_job` chỉ được kích hoạt nếu `test_job` thành công (exit code `0`). Nếu Integration Test thất bại, pipeline dừng ngay lập tức, ngăn mã nguồn lỗi được đóng gói.

2. **Sử dụng `services` để dựng PostgreSQL tạm thời:**
   - Khai báo `services: - name: postgres:14-alpine` yêu cầu GitLab Runner khởi tạo thêm một container PostgreSQL chạy song song và liên kết mạng nội bộ với container Gradle trong suốt quá trình `test_job`.
   - Ứng dụng Spring Boot trong lúc test kết nối tới DB qua hostname `postgres:5432` (cùng `POSTGRES_DB`, `POSTGRES_USER`, `POSTGRES_PASSWORD`). Sau khi job kết thúc, container DB tạm thời tự động bị hủy.

3. **Lưu trữ kết quả build bằng `artifacts`:**
   - Block `artifacts` giúp giữ lại các tệp `.jar` sinh ra tại `build/libs/*.jar` sau khi `build_job` hoàn tất (vì container runner sẽ bị xóa sau mỗi job) và lưu trên GitLab trong `1 day` để tải về hoặc truyền sang stage tiếp theo (như stage `dockerize`).
