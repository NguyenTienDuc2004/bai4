# Bài tập 04 - Tự động đăng bài WordPress bằng n8n + Gemini AI

- **Họ tên**: Nguyễn Tiến Đức
- **MSSV**: K225480106081
- **Email**: K225480106081@tnut.edu.vn
- **Môn học**: Phát triển ứng dụng với mã nguồn mở
- **Giảng viên**: ...

---

## Mô tả bài tập
Xây dựng hệ thống tự động đăng bài lên WordPress thông qua n8n workflow automation:
Telegram Bot → n8n Trigger → Google Gemini AI → JavaScript (parse JSON) → WordPress
Người dùng chỉ cần nhắn tin nội dung chủ đề vào Telegram Bot, hệ thống sẽ tự động:
1. Nhận tin nhắn qua Telegram Trigger
2. Gửi nội dung đến Google Gemini AI để sinh bài viết dạng HTML
3. Parse kết quả JSON lấy tiêu đề và nội dung
4. Tự động đăng bài lên WordPress với trạng thái Publish

---

## Kiến trúc hệ thống

| Service | Image | Chức năng | URL |
|---|---|---|---|
| MariaDB | mariadb:latest | Cơ sở dữ liệu | - |
| phpMyAdmin | phpmyadmin:latest | Quản lý DB | https://k58-pma.nguyentienduc04.io.vn |
| WordPress | wordpress:latest | Blog/Website | https://k58-wp.nguyentienduc04.io.vn |
| Cloudflared | cloudflare/cloudflared | Tunnel ra Internet | - |
| n8n | n8nio/n8n:latest | Workflow automation | https://k58-n8n.nguyentienduc04.io.vn |

---

## Bước 1: Tạo file docker-compose.yml

File docker-compose.yml định nghĩa 5 services chạy cùng nhau trong một network.

 Nội dung file docker-compose.yml (dùng lệnh cat docker-compose.yml) 
<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/303f4d38-9eb5-4437-92cb-6d47ef17dc5d" />
<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/3568f521-8eb7-471a-abad-bea2f002bddc" />

---

## Bước 2: Khởi động các containers

Chạy lệnh `docker compose up -d` để khởi động tất cả services.

 Docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"` showing 5 containers Running
<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/28d25e13-9091-4874-a005-cb02c587fe5b" />

---

## Bước 3: Cấu hình Cloudflare Tunnel

Tạo tunnel **btvn04** và cấu hình 3 Public Hostname để expose các service ra Internet.

| Subdomain | Service nội bộ |
|---|---|
| k58-wp.nguyentienduc04.io.vn | wordpress:80 |
| k58-pma.nguyentienduc04.io.vn | phpmyadmin:80 |
| k58-n8n.nguyentienduc04.io.vn | n8n:5678 |

<img width="1910" height="1072" alt="image" src="https://github.com/user-attachments/assets/5683ce5d-9e6c-455e-a29f-31824c886fc8" />

<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/772452ad-a868-48ff-b276-4140a6eea306" />


---

## Bước 4: Kiểm tra Database với phpMyAdmin

Truy cập `https://k58-pma.nguyentienduc04.io.vn` để kiểm tra database.
Sau khi cài đặt WordPress, hệ thống tự động tạo **12 bảng** `wp_*` trong database `wordpress_db`.

 phpMyAdmin hiển thị 12 bảng wp
<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/e7ee13dc-082a-4389-a2ac-e97e3c3964b7" />

---

## Bước 5: Cài đặt WordPress và tạo bài viết thủ công

Truy cập `https://k58-wp.nguyentienduc04.io.vn` để cài đặt WordPress.

Tạo 2 bài viết thủ công:
- **Bài 1**: Giới thiệu bản thân
- **Bài 2**: Kiến thức học được ở môn Phát triển ứng dụng với mã nguồn mở

Ngoài ra bài viết **"Lợi Ích Không Thể Bỏ Qua Của Docker Trong Phát Triển Phần Mềm"** được tự động đăng bởi n8n (sẽ trình bày ở Bước 7).

<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/1331a5de-e14e-4e35-80ac-1bab8206f1d3" />


---

## Bước 6: Cấu hình n8n Workflow

Truy cập `https://k58-n8n.nguyentienduc04.io.vn` để tạo workflow tự động hóa gồm 4 nodes.

---

### Node 1 - Telegram Trigger
Tạo bot Telegram qua @BotFather với tên `NguyenTienDuc Bot`, username `@NguyenTienDuc_k58_bot`.
Node lắng nghe tin nhắn gửi đến bot, khi có tin nhắn mới sẽ kích hoạt toàn bộ workflow.

<img width="1290" height="2796" alt="image" src="https://github.com/user-attachments/assets/30b1f32c-6d1d-4b20-bde7-34c0bdffe4ab" />


Lắng nghe tin nhắn từ bot `@NguyenTienDuc_k58_bot`, khi có tin nhắn mới sẽ kích hoạt workflow.

<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/7582e6f7-cceb-4e3e-b072-4dd31bc741b3" />


---

### Node 2 - Google Gemini (Message a Model)
- **Model**: `models/gemini-2.5-flash`
- **Prompt**: Nhận nội dung tin nhắn từ Telegram, yêu cầu Gemini sinh bài viết HTML+CSS và trả về JSON với 2 trường `post_title` và `post_content`

<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/97b05161-9297-4d14-b4e4-3cbb1a74bf41" />


---

### Node 3 - Code (JavaScript)
Parse JSON trả về từ Gemini, trích xuất `title` và `content` để truyền sang node WordPress.

```javascript
const rawText = $input.first().json.candidates[0].content.parts[0].text;
const cleanData = JSON.parse(rawText);
return [{ json: { title: cleanData.post_title, content: cleanData.post_content } }];
```
<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/eb5a3c5b-6da1-4c9b-ab27-302bbc1a8a7c" />


---

### Node 4 - WordPress (Create a post)
Tự động tạo bài viết lên WordPress với:
- **Title**: `{{ $json.title }}`
- **Content**: `{{ $json.content }}`
- **Status**: Publish

<img width="1913" height="1079" alt="image" src="https://github.com/user-attachments/assets/89d47ccd-f9f4-4487-8c5a-d442834ab4b4" />


---

### Toàn bộ Workflow

<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/2516ab37-936d-44d0-a93b-b03f9d244793" />


### Kết quả thực thi

<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/bd36500e-39ee-462c-9a3c-d5c74b70d111" />


## Bước 7: Kiểm tra kết quả

Nhắn tin vào bot Telegram: *"Viết bài về lợi ích của Docker trong phát triển phần mềm"*

<img width="1290" height="2796" alt="image" src="https://github.com/user-attachments/assets/95682953-38b4-43cb-be42-5da383b626ae" />

Sau khoảng 20 giây, bài viết tự động xuất hiện trên WordPress:

<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/dbe4b20c-e794-4a4b-8f23-800fd1742a2d" />


<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/3e8f3d81-e787-48d1-ab24-df96bbc9665b" />

## Giới thiệu bản thân

Xin chào! Mình là **Nguyễn Tiến Đức**, sinh viên trường **Đại học Kỹ thuật Công nghiệp - Đại học Thái Nguyên (TNUT)**.

- **Mã sinh viên**: K225480106081
- **Ngành**: Công nghệ Thông tin
- **Email**: K225480106081@tnut.edu.vn

Mình có niềm đam mê với lập trình, tìm hiểu các công nghệ mới và xây dựng các ứng dụng thực tế. Môn **Phát triển ứng dụng với mã nguồn mở** đã giúp mình tiếp cận với nhiều công nghệ hiện đại như Docker, n8n, WordPress, Cloudflare Tunnel và Google Gemini AI.

<img width="1910" height="1068" alt="image" src="https://github.com/user-attachments/assets/699f7e3c-b05f-4542-af62-c38c04bb6fbc" />

---

## Nhận xét & Kết luận

Qua bài tập này, em đã học được:
- **Docker Compose**: Triển khai và quản lý nhiều container cùng lúc, hiểu rõ về volumes, networks và depends_on
- **Cloudflare Tunnel**: Expose ứng dụng nội bộ ra Internet an toàn mà không cần IP tĩnh hay mở port
- **WordPress REST API**: Tích hợp WordPress với các hệ thống bên ngoài thông qua Application Password
- **n8n Workflow Automation**: Xây dựng luồng tự động hóa kết nối nhiều service với nhau
- **Google Gemini AI**: Sử dụng AI để sinh nội dung tự động theo yêu cầu
- **Tích hợp hệ thống**: Kết nối Telegram + AI + WordPress thành một pipeline hoàn chỉnh

Hệ thống hoạt động ổn định, chỉ cần nhắn một tin nhắn là có ngay bài viết chất lượng được đăng tự động lên website.
