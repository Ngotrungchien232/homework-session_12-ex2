# Bài 2: Tự động hóa đóng gói và phát hành Docker Image khi gắn Git Tag (`v*.*.*`)

## 1. Mục tiêu
- Cấu hình luồng CD tự động phát hành phiên bản Docker Image chính thức khi có sự kiện gán nhãn Release (**Git Tag `v*.*.*`**).
- Tạo thẻ Docker Tag tự động tương ứng với số phiên bản Git Tag bằng `docker/metadata-action@v5`.
- Đóng gói và đẩy Docker Image lên **GitHub Container Registry (`ghcr.io`)** bằng `docker/build-push-action@v5`.

---

## 2. Cấu trúc thư mục dự án

```text
SESSION12_BAI2/
├── .github/
│   └── workflows/
│       └── release.yml           # Kịch bản CD đóng gói & phát hành Docker Image theo Git Tag
├── gradle/
│   └── wrapper/
│       ├── gradle-wrapper.jar
│       └── gradle-wrapper.properties
├── homework/
│   └── session_12/
│       └── ex2/                  # Thư mục sản phẩm nộp bài theo đường dẫn gợi ý
│           ├── .github/
│           │   └── workflows/
│           │       └── release.yml
│           └── README.md
├── src/
│   ├── main/java/com/example/
│   │   ├── Application.java
│   │   └── Calculator.java
│   └── test/java/com/example/
│       └── CalculatorTest.java
├── .dockerignore
├── .gitattributes
├── .gitignore
├── Dockerfile                    # Multi-stage Dockerfile đóng gói ứng dụng
├── build.gradle
├── gradlew
├── gradlew.bat
├── settings.gradle
└── README.md
```

---

## 3. Nội dung tệp kịch bản `.github/workflows/release.yml`

```yaml
name: Release Docker Image to GHCR

on:
  push:
    tags:
      - 'v*.*.*'

jobs:
  release-image:
    name: Build and Release Docker Image
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write

    steps:
      - name: Checkout repository
        uses: actions/checkout@v5

      - name: Log in to GitHub Container Registry (GHCR)
        uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Extract Docker metadata (tags, labels) from Git
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: ghcr.io/${{ github.repository }}
          tags: |
            type=ref,event=tag
            type=semver,pattern={{version}}
            type=raw,value=latest

      - name: Build and push Docker image to GHCR
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
```

---

## 4. Hướng dẫn kiểm tra và nộp bài

### Bước 1: Đẩy mã nguồn lên GitHub Repository
Mở terminal tại thư mục `SESSION12_BAI2` và thực hiện:

```bash
git init
git add .
git commit -m "feat(ex2): add CD workflow release.yml for Git Tag Docker release"
git branch -M main
git remote add origin <URL_GITHUB_REPO_CUA_BAN>
git push -u origin main
```

### Bước 2: Gán Git Tag `v1.0.0` và đẩy lên GitHub để kích hoạt Release
Thực thi lệnh gán tag local và đẩy lên GitHub theo đúng yêu cầu đề bài:

```bash
git tag v1.0.0 && git push origin v1.0.0
```

### Bước 3: Kiểm tra kết quả và chụp ảnh màn hình nộp bài
1. Truy cập vào **tab Actions** trên GitHub Repository:
   - Quan sát workflow **Release Docker Image to GHCR** (`release.yml`) tự động được kích hoạt bởi sự kiện push tag `v1.0.0` và chạy thành công (tích xanh).
2. Truy cập vào trang chính của Repository, nhìn sang cột bên phải mục **Packages** (hoặc vào trang cá nhân -> tab **Packages**):
   - Nhấn vào gói package vừa được tạo.
   - Kiểm tra sự xuất hiện của gói Docker Image `ghcr.io/<owner>/<repo>:v1.0.0`.
3. **Chụp ảnh màn hình** gói Docker Image `v1.0.0` trên **GitHub Packages** và lưu vào thư mục `homework/session_12/ex2/` để hoàn tất nộp bài.
