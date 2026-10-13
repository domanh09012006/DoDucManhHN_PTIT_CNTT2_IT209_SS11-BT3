# Báo cáo so sánh thời gian chạy CI bằng Gradle Dependency Caching

## 1. Thông tin tổng quan
- **Dự án:** Spring Boot (Gradle)
- **Workflow:** `.github/workflows/ci.yml`
- **Cấu hình Cache:** `cache: 'gradle'` trong bước `actions/setup-java@v5`
- **Người thực hiện:** `Cao Chí Thiện`
- **Ngày thực hiện:** `12-10-2026`

---

## 2. Bảng so sánh thời gian thực thi (GitHub Actions)

| Tiêu chí / Bước thực hiện | Lần 1 (Cache Miss - Khởi tạo cache) | Lần 2 (Cache Hit - Tái sử dụng cache) | Chênh lệch / Cải thiện |
| :--- | :--- | :--- | :--- |
| **Commit SHA / Run ID** | `5939aa3 ` | `72bc266` | - |
| **Trạng thái Cache (Setup Java)** | `Cache not found / miss` | `Resolved from cache / hit` | - |
| **Thời gian: Checkout mã nguồn** | `0` s | `1` s | `+1` s |
| **Thời gian: Setup Java (temurin 17)** | `1` s | `4` s | `+3` s |
| **Thời gian: `./gradlew test`** | `39` s | `9` s | `-30` s |
| **Thời gian: `./gradlew bootJar`** | `2` s | `1` s | `-1` s |
| **Thời gian: Tải artifact (app-jar)** | `1` s | `2` s | `+1` s |
| **Tổng thời gian chạy Workflow** | **`54` s** | **`25` s** | **Giảm `53.7`%** |

---

## 3. Nhật ký kiểm tra Cache trên GitHub Actions Logs

### Lần chạy 1:
- **Log bước `actions/setup-java@v5`:**
  ```text
Run actions/setup-java@v5
Installed distributions
Creating settings.xml with server-id: github
Writing to /home/runner/.m2/settings.xml
Configured MAVEN_ARGS to include -ntp to suppress Maven transfer progress logs. Set 'show-download-progress: true' to keep the download progress output.
gradle cache is not found

Post job cleanup.
/usr/bin/tar --posix -cf cache.tzst --exclude cache.tzst -P -C /home/runner/work/PTIT_CNTT2_IT209-Session11-Bai3/PTIT_CNTT2_IT209-Session11-Bai3 --files-from manifest.txt --use-compress-program zstdmt
Sent 16467169 of 150684897 (10.9%), 15.7 MBs/sec
Sent 150684897 of 150684897 (100.0%), 117.9 MBs/sec
gradle-wrapper cache saved with the key: setup-java-Linux-x64-gradle-wrapper-cb10f63b6e80d04c47ba63d0097b1be1d92513baebbf33fb9b91e65214965f4c
/usr/bin/tar --posix -cf cache.tzst --exclude cache.tzst -P -C /home/runner/work/PTIT_CNTT2_IT209-Session11-Bai3/PTIT_CNTT2_IT209-Session11-Bai3 --files-from manifest.txt --use-compress-program zstdmt
Sent 18939904 of 41582689 (45.5%), 18.1 MBs/sec
Sent 41582689 of 41582689 (100.0%), 20.5 MBs/sec
Cache saved with the key: setup-java-Linux-x64-gradle-2d94d23d757f8be5f872ec31499f07d55b835a21c7d841ba90b175ed1e7e614d
  ```

### Lần chạy 2:
- **Log bước `actions/setup-java@v5`:**
  ```text
Run actions/setup-java@v5
Installed distributions
Creating settings.xml with server-id: github
Writing to /home/runner/.m2/settings.xml
Configured MAVEN_ARGS to include -ntp to suppress Maven transfer progress logs. Set 'show-download-progress: true' to keep the download progress output.
Cache hit for: setup-java-Linux-x64-gradle-2d94d23d757f8be5f872ec31499f07d55b835a21c7d841ba90b175ed1e7e614d
Received 41582689 of 41582689 (100.0%), 41.5 MBs/sec
Cache Size: ~40 MB (41582689 B)
/usr/bin/tar -xf /home/runner/work/_temp/a818ee92-9ff0-4980-bbc7-3a7550764351/cache.tzst -P -C /home/runner/work/PTIT_CNTT2_IT209-Session11-Bai3/PTIT_CNTT2_IT209-Session11-Bai3 --use-compress-program unzstd
Cache restored successfully
Cache restored from key: setup-java-Linux-x64-gradle-2d94d23d757f8be5f872ec31499f07d55b835a21c7d841ba90b175ed1e7e614d
Cache hit for: setup-java-Linux-x64-gradle-wrapper-cb10f63b6e80d04c47ba63d0097b1be1d92513baebbf33fb9b91e65214965f4c
Received 50331648 of 150684897 (33.4%), 48.0 MBs/sec
Received 150684897 of 150684897 (100.0%), 88.7 MBs/sec
Cache Size: ~144 MB (150684897 B)
/usr/bin/tar -xf /home/runner/work/_temp/ed0f1b8a-7292-416e-8079-5c0bc99b2299/cache.tzst -P -C /home/runner/work/PTIT_CNTT2_IT209-Session11-Bai3/PTIT_CNTT2_IT209-Session11-Bai3 --use-compress-program unzstd
Cache restored successfully
gradle-wrapper cache restored from key: setup-java-Linux-x64-gradle-wrapper-cb10f63b6e80d04c47ba63d0097b1be1d92513baebbf33fb9b91e65214965f4c

Post job cleanup.
Cache hit occurred on the gradle-wrapper primary key setup-java-Linux-x64-gradle-wrapper-cb10f63b6e80d04c47ba63d0097b1be1d92513baebbf33fb9b91e65214965f4c, not saving cache.
Cache hit occurred on the primary key setup-java-Linux-x64-gradle-2d94d23d757f8be5f872ec31499f07d55b835a21c7d841ba90b175ed1e7e614d, not saving cache.
  ```

---

## 4. Nhận xét & Đánh giá kết quả thực tế
- **Cơ chế hoạt động:**
  - Ở lần chạy đầu tiên, các dependencies của Gradle được tải về từ remote repository và sau đó được lưu vào GitHub Actions Cache storage.
  - Ở lần chạy tiếp theo, `actions/setup-java@v5` phục hồi cache dependencies giúp bỏ qua việc tải lại toàn bộ dependencies, từ đó rút ngắn thời gian build.
- **Đánh giá hiệu quả thực tế:**
  - *Hiệu quả tốc độ:* Tổng thời gian chạy workflow giảm mạnh từ 54 giây xuống còn 25 giây, tương đương mức giảm ~53.7% tổng thời gian thực thi. Riêng bước chạy lệnh kiểm thử (./gradlew test) được rút ngắn đáng kể từ 39 giây xuống còn 9 giây.
  - *Mức độ ổn định:* Tính năng caching giúp pipeline hoạt động mượt mà, độc lập với tốc độ mạng bên ngoài và tăng hiệu suất phát triển phần mềm đáng kể.
