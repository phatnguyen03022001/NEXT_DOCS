# I. Giới thiệu và Cài đặt

## 1. Next.js là gì? Ưu điểm và nhược điểm

### Next.js là gì?

Next.js là một framework mã nguồn mở được xây dựng trên React, hỗ trợ phát triển ứng dụng web hiệu suất cao với các tính năng như **Server-Side Rendering (SSR)**, **Static Site Generation (SSG)**, và **Client-Side Rendering (CSR)**. Next.js cung cấp cấu trúc dự án mạnh mẽ, giúp phát triển các ứng dụng web nhanh, tối ưu SEO, và dễ mở rộng.

### Ưu điểm:

- **Hiệu suất cao**: Tự động tối ưu hóa với SSR, SSG, và Incremental Static Regeneration (ISR).
- **SEO thân thiện**: Tạo HTML ngay từ phía server, hỗ trợ công cụ tìm kiếm.
- **Tích hợp sẵn**: Cung cấp routing, API routes, và tối ưu hóa hình ảnh (Image Optimization).
- **Hỗ trợ mạnh mẽ**: Cộng đồng lớn, được Vercel phát triển và hỗ trợ triển khai dễ dàng.
- **Tính linh hoạt**: Hỗ trợ cả SSR, SSG, và CSR, phù hợp với nhiều loại dự án.
- **TypeScript tích hợp**: Hỗ trợ TypeScript mà không cần cấu hình phức tạp.

### Nhược điểm:

- **Khúc học tập**: Khá phức tạp với những người mới học React hoặc chưa quen với SSR/SSG.
- **Hạn chế tùy chỉnh**: Một số tính năng bị giới hạn bởi cấu trúc framework.
- **Phụ thuộc vào Vercel**: Một số tính năng tối ưu hóa hoạt động tốt nhất khi triển khai trên Vercel.
- **Chi phí server**: SSR có thể tốn tài nguyên server nếu lưu lượng truy cập lớn.

## 2. So sánh với React CRA và các framework khác

| **Tiêu chí**         | **Next.js**                          | **React CRA**                        | **Gatsby**              | **Nuxt.js (Vue)**             |
| -------------------- | ------------------------------------ | ------------------------------------ | ----------------------- | ----------------------------- |
| **Framework**        | Dựa trên React                       | Thư viện React                       | Dựa trên React          | Dựa trên Vue                  |
| **Rendering**        | SSR, SSG, CSR, ISR                   | Chỉ CSR                              | Chủ yếu SSG, hỗ trợ SSR | SSR, SSG                      |
| **SEO**              | Rất tốt (SSR/SSG)                    | Kém (chỉ CSR)                        | Tốt (SSG)               | Tốt (SSR/SSG)                 |
| **Routing**          | File-based routing                   | Cần cài thêm thư viện (React Router) | File-based routing      | File-based routing            |
| **API Routes**       | Tích hợp sẵn                         | Không hỗ trợ                         | Không hỗ trợ            | Tích hợp sẵn                  |
| **Tối ưu hiệu suất** | Tự động (code splitting, image opt.) | Cần cấu hình thủ công                | Tốt (tối ưu SSG)        | Tốt (tối ưu SSR/SSG)          |
| **Độ phức tạp**      | Trung bình                           | Đơn giản                             | Trung bình              | Trung bình                    |
| **Triển khai**       | Dễ dàng (Vercel, Netlify,...)        | Cần cấu hình thêm                    | Dễ dàng (Netlify,...)   | Dễ dàng (Vercel, Netlify,...) |

**Kết luận**:

- **Next.js**: Phù hợp cho các dự án cần SEO, hiệu suất cao, và tích hợp API.
- **React CRA**: Tốt cho ứng dụng đơn giản, chỉ cần CSR, hoặc học tập React.
- **Gatsby**: Lý tưởng cho trang tĩnh, blog, hoặc trang cần tốc độ tải nhanh.
- **Nuxt.js**: Tương tự Next.js nhưng dành cho Vue, phù hợp nếu bạn đã quen với Vue.

## 3. Cài đặt môi trường phát triển

Để bắt đầu với Next.js, bạn cần chuẩn bị môi trường phát triển:

### Yêu cầu:

- **Node.js**: Phiên bản 18.x hoặc cao hơn (khuyến nghị dùng phiên bản LTS mới nhất).
- **npm** hoặc **yarn**: Quản lý gói (npm đi kèm với Node.js).
- **Code editor**: VS Code, WebStorm, hoặc bất kỳ trình soạn thảo nào hỗ trợ JavaScript/TypeScript.

### Cài đặt Node.js:

1. Tải và cài Node.js từ nodejs.org.

2. Kiểm tra phiên bản:

   ```bash
   node -v
   npm -v
   ```

### Cài đặt thêm (tùy chọn):

- **Yarn** (nếu muốn dùng thay npm):

  ```bash
  npm install -g yarn
  ```

- **TypeScript** (tích hợp sẵn trong Next.js, nhưng có thể cài global):

  ```bash
  npm install -g typescript
  ```

## 4. Tạo project Next.js đầu tiên (npx create-next-app)

1. **Tạo dự án**: Chạy lệnh sau để tạo một dự án Next.js mới:

   ```bash
   npx create-next-app@latest my-next-app
   ```

   - `my-next-app`: Tên thư mục dự án.
   - Lệnh sẽ hỏi một số tùy chọn (TypeScript, ESLint, Tailwind CSS,...). Chọn theo nhu cầu hoặc để mặc định.

2. **Di chuyển vào thư mục dự án**:

   ```bash
   cd my-next-app
   ```

3. **Chạy dự án**:

   ```bash
   npm run dev
   ```

   Mở trình duyệt tại `http://localhost:3000` để xem ứng dụng.

4. **Tùy chọn nâng cao**:

   - Nếu muốn dùng TypeScript hoặc các tính năng cụ thể, bạn có thể cấu hình thêm khi tạo dự án:

     ```bash
     npx create-next-app@latest --typescript
     ```

## 5. Cấu trúc thư mục mặc định trong Next.js

Khi tạo dự án với `create-next-app`, cấu trúc thư mục mặc định như sau:

```
my-next-app/
├── node_modules/            # Thư viện được cài đặt
├── public/                 # Tài nguyên tĩnh (hình ảnh, favicon,...)
│   ├── favicon.ico
│   └── images/
├── src/                    # Thư mục mã nguồn chính (tùy chọn, có thể không có nếu dùng root)
│   ├── app/                # Hệ thống routing chính (App Router, Next.js 13+)
│   │   ├── page.js         # Trang chủ (localhost:3000)
│   │   ├── layout.js       # Layout chính cho ứng dụng
│   │   └── [folder]/       # Các route động hoặc tĩnh
│   ├── components/         # Các component React tái sử dụng
│   └── styles/             # CSS hoặc Tailwind CSS
├── .gitignore              # Cấu hình file/thư mục bỏ qua khi dùng Git
├── next.config.js          # Cấu hình Next.js
├── package.json            # Quản lý dependencies và scripts
├── README.md               # Tài liệu dự án
└── tsconfig.json           # Cấu hình TypeScript (nếu dùng)
```

### Giải thích các thư mục/file chính:

- [ ] `public/`: Lưu trữ tài nguyên tĩnh như hình ảnh, favicon. Có thể truy cập trực tiếp qua `/tên_file` (VD: `/favicon.ico`).

  - `src/app/`: Sử dụng App Router (Next.js 13+), định nghĩa các route thông qua cấu trúc thư mục. Mỗi thư mục tương ứng với một route.
  - `page.js`: File định nghĩa nội dung của một trang (VD: `app/page.js` là trang chủ).
  - `layout.js`: File định nghĩa layout chung cho ứng dụng hoặc một nhóm route.
  - `next.config.js`: File cấu hình tùy chỉnh (VD: cấu hình môi trường, redirect, rewrite).

`package.json`: Quản lý dependencies và scripts (VD: `npm run dev`, `npm run build`).