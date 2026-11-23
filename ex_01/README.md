# Bài tập 1 (SS09): Khắc phục sự cố Pipeline CI/CD cơ bản

## 1. Phân tích nguyên nhân gây lỗi

Cấu hình `.gitlab-ci.yml` ban đầu:
```yaml
stages:
  build_app

build_job:
  stage: build_app
  script:
    - ./gradlew clean build -x test
```

Khi chạy trên GitLab Runner mặc định (Docker Executor), pipeline gặp lỗi ngay từ bước khởi tạo do **3 nguyên nhân/thiếu sót sau**:

1. **Thiếu định nghĩa môi trường thực thi (`image:`):**
   - Trong file `.gitlab-ci.yml` không khai báo từ khóa `image:`.
   - Khi GitLab Runner chạy với Docker Executor, nó sẽ sử dụng default image tối giản (ví dụ `ruby` hoặc `alpine`) — **không cài đặt sẵn Java (JDK 17) hay Gradle**.
   - Kết quả: Khi thực thi `./gradlew clean build -x test`, shell hoặc Gradle Wrapper tìm lệnh `java` không thấy và lập tức báo lỗi **`command not found`** (hoặc `JAVA_HOME is not set`).

2. **Sai cú pháp YAML khi khai báo `stages`:**
   - Từ khóa `stages` trong GitLab CI bắt buộc phải là một **mảng (YAML List)** sử dụng dấu gạch đầu dòng `-`.
   - Cấu hình cũ viết `stages: build_app` (dạng scalar/chuỗi đơn), khiến GitLab CI Parser báo lỗi cú pháp YAML (`stages should be an array of strings`) và không map được với `stage: build_app` trong `build_job`.

3. **Rủi ro quyền thực thi đối với `./gradlew`:**
   - Khi code được push từ Windows hoặc clone lại, file `gradlew` có thể bị mất quyền thực thi (`+x`), gây ra lỗi `Permission denied` nếu không chạy `chmod +x gradlew` trong `before_script`.

---

## 2. File `.gitlab-ci.yml` đã sửa hoàn chỉnh

Xem trực tiếp tại: [`.gitlab-ci.yml`](./.gitlab-ci.yml)

```yaml
image: gradle:7.6-jdk17

stages:
  - build_app

cache:
  key: "$CI_COMMIT_REF_SLUG"
  paths:
    - .gradle/wrapper
    - .gradle/caches

build_job:
  stage: build_app
  before_script:
    - chmod +x gradlew
  script:
    - ./gradlew clean build -x test
```

### Giải thích các thay đổi:
- **`image: gradle:7.6-jdk17`**: Cung cấp container môi trường chuẩn chứa sẵn **JDK 17** và **Gradle 7.6**, giúp `./gradlew` chạy mượt mà trên mọi Docker Runner.
- **`stages: - build_app`**: Sửa thành mảng YAML hợp lệ với dấu `-`.
- **`before_script: - chmod +x gradlew`**: Cấp quyền thực thi cho Gradle Wrapper trước khi chạy build.
- **`cache:`**: Lưu cache thư viện Gradle giữa các lần chạy pipeline để tăng tốc độ build.
