# OmniRoute - Sổ Tay Cá Nhân (thai-coder)

> **Mục đích:** Hướng dẫn quy trình build cục bộ trên máy tính để tránh lỗi sập RAM trên Vercel, cùng cơ chế bảo đảm an toàn khi đẩy code lên GitHub Fork cá nhân.

---

## 1. Lưu Ý An Toàn Về Cơ Chế Fork & Remote (Không Ảnh Hưởng Chủ Sở Hữu)

### Cơ chế phân quyền độc lập của GitHub Fork:

- Khi bạn fork một dự án, GitHub sẽ tạo ra một bản sao hoàn toàn tách biệt thuộc quyền sở hữu của tài khoản `thai-coder`.
- Mọi thao tác `git push`, `commit`, xóa hay sửa đổi nhánh trên `https://github.com/thai-coder/OmniRoute.git` chỉ lưu trên tài khoản của bạn.
- Kho gốc của tác giả chỉ bị ảnh hưởng **duy nhất** khi bạn chủ động tạo một **Pull Request (PR)** gửi sang cho họ và họ tự tay bấm nút **Merge**. Nếu bạn không tạo PR, tác giả gốc thậm chí không nhận bất kỳ thay đổi nào từ bạn.

### Cấu hình remote trên máy của bạn hiện tại:

- **`origin`**: Trỏ về `https://github.com/thai-coder/OmniRoute.git` (đây là nơi bạn thực hiện `git push` để lưu mã nguồn và deploy).
- **`upstream`**: Trỏ về `https://github.com/cheaper-inference/OmniRoute.git` (chỉ dùng nếu sau này bạn muốn kéo thêm các tính năng mới từ bản gốc về bằng `git fetch upstream`).

---

## 2. Quy Trình 6 Bước Build Cục Bộ & Deploy Lên Vercel

Để đưa toàn bộ quy trình này về xử lý cục bộ trên máy tính cá nhân và đẩy thẳng lên Vercel, bạn mở Terminal (hoặc Command Prompt/PowerShell) và chạy tuần tự các lệnh sau:

### Bước 1: Tải mã nguồn và cài đặt thư viện lõi

```bash
git clone https://github.com/thai-coder/OmniRoute.git
cd OmniRoute
npm install
```

### Bước 2: Cài đặt công cụ Vercel CLI toàn cục

```bash
npm install -g vercel
```

### Bước 3: Đăng nhập và liên kết dự án

```bash
vercel login
```

_(Hệ thống sẽ mở trình duyệt để bạn xác thực tài khoản Vercel hoặc GitHub)._  
Sau khi đăng nhập thành công, gõ tiếp:

```bash
vercel link
```

_(Nhấn Enter liên tục để đồng ý thiết lập thư mục hiện tại và chọn đúng dự án `omni-route` đã có sẵn trên Vercel)._

### Bước 4: Đồng bộ môi trường (Rất quan trọng)

Bạn cần kéo các biến môi trường từ Vercel về máy tính để quá trình biên dịch không bị báo lỗi thiếu cấu hình:

```bash
vercel pull --yes --environment=production
```

### Bước 5: Kích hoạt máy tính cày cuốc (Build cục bộ)

```bash
vercel build --prod
```

_(Đây là lúc PC của bạn sẽ dùng toàn bộ lượng RAM 16GB/32GB để xử lý các thư viện nặng như onnxruntime-node hay playwright. Quá trình này sẽ tạo ra thư mục `.vercel/output` chứa các file đã được tối ưu tĩnh)._

### Bước 6: Bắn thành phẩm lên Vercel

Khi lệnh build chạy xong và báo thành công, bạn đẩy bản đóng gói nhẹ tênh đó lên máy chủ:

```bash
vercel deploy --prebuilt --prod
```

---

## 3. Tối Ưu Hóa Quy Trình Với Lệnh Tắt (NPM Scripts)

Để không cần phải nhớ hay gõ tuần tự nhiều lệnh, dự án đã được tích hợp sẵn 2 phím tắt trong `package.json`:

```json
"scripts": {
  "vercel:deploy": "vercel build --prod && vercel deploy --prebuilt --prod",
  "vercel:update": "git pull && npm install && npm run vercel:deploy"
}
```

### 🚀 Lệnh 1: `npm run vercel:deploy`

- **Tác dụng:** Dùng khi bạn vừa tự tay chỉnh sửa code trên máy tính và muốn đẩy thẳng lên web.
- **Cơ chế:** Hệ thống tự động chạy `vercel build --prod`. Ký hiệu `&&` đảm bảo chỉ khi build thành công 100%, lệnh `vercel deploy --prebuilt --prod` mới được kích hoạt để bắn bản build lên Vercel.

### ☕ Lệnh 2: `npm run vercel:update`

- **Tác dụng:** Dùng khi muốn đồng bộ phiên bản mới nhất từ GitHub lên web chỉ với **1 lệnh duy nhất**.
- **Cơ chế:**
  1. Chạy `git pull` để lấy toàn bộ code mới nhất về máy tính.
  2. Chạy `npm install` để cập nhật các thư viện mới (nếu có).
  3. Tự động gọi tiếp `npm run vercel:deploy` để build cục bộ và deploy lên Vercel.

---

> **Kết luận:** Toàn bộ gánh nặng phần cứng giờ đây đã do máy tính cá nhân của bạn chịu trách nhiệm. Máy chủ Vercel chỉ việc nhận kết quả cuối cùng để phát sóng website ra internet. Quá trình deploy của bạn sẽ luôn ổn định và không bao giờ gặp lại lỗi sập RAM nữa.
