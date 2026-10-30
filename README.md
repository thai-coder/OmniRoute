# OmniRoute - Hướng dẫn Build Cục Bộ & Deploy Vercel

Để đưa toàn bộ quy trình này về xử lý cục bộ trên máy tính cá nhân và đẩy thẳng lên Vercel, bạn mở Terminal (hoặc Command Prompt/PowerShell) và chạy tuần tự các lệnh sau:

1. **Tải mã nguồn và cài đặt thư viện lõi:**

```bash
git clone https://github.com/thai-coder/OmniRoute.git
cd OmniRoute
npm install
```

2. **Cài đặt công cụ Vercel CLI toàn cục:**

```bash
npm install -g vercel
```

3. **Đăng nhập và liên kết dự án:**

```bash
vercel login
```

_(Hệ thống sẽ mở trình duyệt để bạn xác thực tài khoản Vercel hoặc GitHub)._
Sau khi đăng nhập thành công, gõ tiếp:

```bash
vercel link
```

_(Nhấn Enter liên tục để đồng ý thiết lập thư mục hiện tại và chọn đúng dự án `omni-route` đã có sẵn trên Vercel)._

4. **Đồng bộ môi trường (Rất quan trọng):**
   Bạn cần kéo các biến môi trường từ Vercel về máy tính để quá trình biên dịch không bị báo lỗi thiếu cấu hình:

```bash
vercel pull --yes --environment=production
```

5. **Kích hoạt máy tính cày cuốc (Build cục bộ):**

```bash
vercel build --prod
```

_(Đây là lúc PC của bạn sẽ dùng toàn bộ lượng RAM 16GB/32GB để xử lý các thư viện nặng như onnxruntime-node hay playwright. Quá trình này sẽ tạo ra thư mục `.vercel/output` chứa các file đã được tối ưu tĩnh)._

6. **Bắn thành phẩm lên Vercel:**
   Khi lệnh build chạy xong và báo thành công, bạn đẩy bản đóng gói nhẹ tênh đó lên máy chủ:

```bash
vercel deploy --prebuilt --prod
```

Toàn bộ gánh nặng phần cứng giờ đây đã do máy tính cá nhân của bạn chịu trách nhiệm. Máy chủ Vercel chỉ việc nhận kết quả cuối cùng ở bước 6 để phát sóng website ra internet. Quá trình deploy của bạn sẽ luôn ổn định và không bao giờ gặp lại lỗi sập RAM nữa.
