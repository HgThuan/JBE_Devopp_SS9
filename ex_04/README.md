# Bài tập 4 (SS09): Đẩy Docker Image lên Docker Hub thông qua GitLab CI

## 1. File `.gitlab-ci.yml` hoàn chỉnh (gồm stage `dockerize`)

Xem trực tiếp tại: [`.gitlab-ci.yml`](./.gitlab-ci.yml) và [`Dockerfile`](./Dockerfile)

### Chi tiết stage `dockerize_job`:
```yaml
dockerize_job:
  stage: dockerize
  image: docker:latest
  services:
    - docker:dind
  variables:
    DOCKER_HOST: tcp://docker:2376
    DOCKER_TLS_CERTDIR: "/certs"
    IMAGE_NAME: "payment-service"
    IMAGE_TAG: "1.0.0"
  dependencies:
    - build_job
  before_script:
    - echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin
  script:
    - docker build -t "$DOCKER_USERNAME/$IMAGE_NAME:$IMAGE_TAG" -t "$DOCKER_USERNAME/$IMAGE_NAME:latest" .
    - docker push "$DOCKER_USERNAME/$IMAGE_NAME:$IMAGE_TAG"
    - docker push "$DOCKER_USERNAME/$IMAGE_NAME:latest"
  after_script:
    - docker logout
```

---

## 2. Cơ chế hoạt động & Bảo mật

1. **Docker-in-Docker (DinD):**
   - Dùng `image: docker:latest` kết hợp `services: - docker:dind` để khởi chạy một Docker Daemon cô lập phục vụ build và push image ngay trong pipeline GitLab CI.
   - `DOCKER_TLS_CERTDIR: "/certs"` bật mã hóa TLS giữa Docker CLI và DinD service.
2. **Truyền file `.jar` giữa các Stage (`dependencies` + `artifacts`):**
   - `dockerize_job` khai báo `dependencies: - build_job` để GitLab Runner tự động tải về thư mục `build/libs/*.jar` đã được lưu artifact ở stage `build`.
3. **Bảo mật thông tin đăng nhập Docker Hub:**
   - Tuyệt đối không hard-code tài khoản vào file YAML.
   - Sử dụng `$DOCKER_USERNAME` và `$DOCKER_PASSWORD` từ **GitLab CI/CD Variables** cùng cờ `--password-stdin` để tránh lộ password trong log console.

---

## 3. Hướng dẫn thiết lập trên GitLab & Docker Hub để hoàn thiện bài nộp trên Portal

Để chạy thực tế pipeline trên GitLab và lấy ảnh chụp nộp Portal, bạn thực hiện 3 bước sau trên tài khoản cá nhân:

1. **Trên Docker Hub (https://hub.docker.com):**
   - Tạo repository trống tên `payment-service` (hoặc tên tương ứng với `IMAGE_NAME`).
   - Tạo Access Token trong **Account Settings -> Security -> New Access Token** (dùng làm `DOCKER_PASSWORD`).
2. **Trên GitLab Project:**
   - Vào **Settings -> CI/CD -> Variables -> Add variable**:
     - Key: `DOCKER_USERNAME` | Value: `<username_dockerhub_của_bạn>`
     - Key: `DOCKER_PASSWORD` | Value: `<docker_hub_access_token>` (tích chọn **Mask variable** để che mật khẩu trên log).
   - Chụp ảnh màn hình danh sách **CI/CD Variables** này để nộp kèm bài.
3. **Push code lên GitLab & kiểm tra:**
   - Khi pipeline 3 stage (`test` -> `build` -> `dockerize`) chạy xanh, lấy đường link tới Docker Hub Repository (`https://hub.docker.com/r/<username>/payment-service`) nộp lên Portal.
