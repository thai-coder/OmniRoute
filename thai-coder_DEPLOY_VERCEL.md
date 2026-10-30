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

## 4. Lỗi `NEXT_OUTPUT_DIR_MISSING` và cách đã sửa

**Triệu chứng:** `vercel build --prod` build xong Next.js nhưng báo _"The Next.js output directory .next was not found"_.

**Nguyên nhân:** `next.config.mjs` ghi output ra `.build/next` (biến `NEXT_DIST_DIR`) và mặc định bật `output: "standalone"` (dành cho Docker/VPS). Vercel chỉ tìm ở `.next`.

**Cách sửa:** thêm 2 biến môi trường vào Vercel project (môi trường Production):

| Biến                        | Giá trị | Tác dụng                               |
| --------------------------- | ------- | -------------------------------------- |
| `NEXT_DIST_DIR`             | `.next` | Ghi output vào thư mục Vercel mong đợi |
| `OMNIROUTE_SKIP_STANDALONE` | `1`     | Tắt đóng gói standalone                |

### Thêm bằng CLI (từ thư mục dự án)

```bash
vercel env add NEXT_DIST_DIR --environment production --value .next --no-sensitive --yes
vercel env add OMNIROUTE_SKIP_STANDALONE --environment production --value 1 --no-sensitive --yes
vercel env ls --environment production
```

Hoặc thêm trên web: Vercel → Project `omni-route` → Settings → Environment Variables.

> ⚠️ **Bắt buộc dùng `--no-sensitive` (kiểu Config), không để kiểu Secret/Sensitive.** Biến Sensitive khi `vercel pull` chỉ trả về chuỗi `[SENSITIVE]`, build sẽ ghi output vào thư mục tên `[SENSITIVE]`. Nếu đã lỡ tạo, xóa thư mục `[SENSITIVE]` ở gốc dự án và thêm lại bằng `--force`.

### Kiểm tra sau khi `vercel pull`

```bash
vercel pull --yes --environment=production
grep -n "NEXT_DIST_DIR\|OMNIROUTE_SKIP" .vercel/.env.production.local
# phải thấy: NEXT_DIST_DIR=".next" và OMNIROUTE_SKIP_STANDALONE="1"
```

### Script `npm run vercel:deploy`

Script đã gồm cả bước `vercel pull`, nên biến môi trường luôn được cập nhật trước khi build:

```json
"vercel:deploy": "vercel pull --yes --environment=production && vercel build --prod && vercel deploy --prebuilt --prod"
```

### GitHub Actions

`.github/workflows/deploy-vercel.yml` đã được viết đầy đủ (chạy khi push lên `main` hoặc bấm tay ở tab Actions → Deploy to Vercel → Run workflow). Nó chạy `vercel pull` trước `vercel build` nên tự nhận 2 biến trên.

Cần tạo 3 secret ở GitHub: repo `thai-coder/OmniRoute` → Settings → Secrets and variables → Actions → New repository secret:

| Secret              | Lấy ở đâu                                |
| ------------------- | ---------------------------------------- |
| `VERCEL_TOKEN`      | vercel.com/account/tokens → Create Token |
| `VERCEL_ORG_ID`     | `orgId` trong `.vercel/project.json`     |
| `VERCEL_PROJECT_ID` | `projectId` trong `.vercel/project.json` |

### Lưu ý khi chạy

Ứng dụng dùng SQLite (`better-sqlite3`), worker và child process. Serverless của Vercel có hệ thống file chỉ đọc nên build qua được vẫn có thể lỗi khi chạy. Nếu gặp lỗi runtime, VPS/Docker phù hợp hơn.

---

> **Kết luận:** Toàn bộ gánh nặng phần cứng giờ đây đã do máy tính cá nhân của bạn chịu trách nhiệm. Máy chủ Vercel chỉ việc nhận kết quả cuối cùng để phát sóng website ra internet. Quá trình deploy của bạn sẽ luôn ổn định và không bao giờ gặp lại lỗi sập RAM nữa.
