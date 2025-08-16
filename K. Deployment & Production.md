# X. Deployment & Production trong Next.js

## 1. Build và export project (next build, next export)

- **Mục đích**: Chuẩn bị ứng dụng Next.js cho môi trường production bằng cách build và export (nếu cần) thành các file tĩnh hoặc server-side.
- **next build**:
  - Tạo phiên bản tối ưu hóa của ứng dụng cho production.
  - Tạo thư mục `.next` chứa các file được biên dịch (HTML, JS, CSS, v.v.).
  - Lệnh: `npm run build`
- **next export**:
  - Xuất ứng dụng thành các file HTML tĩnh để triển khai trên các nền tảng không yêu cầu server Node.js.
  - Chỉ hoạt động với các trang sử dụng Static Site Generation (SSG) hoặc không có `getServerSideProps`.
  - Lệnh: `npm run export`
  - Kết quả: Tạo thư mục `out` chứa các file HTML tĩnh.
- **Ví dụ**:
  - Cấu hình `package.json`:
```json
{
  "scripts": {
    "build": "next build",
    "export": "next export",
    "start": "next start"
  }
}
```
  - Chạy:
    - `npm run build` để tạo build production.
    - `npm run export` để xuất file tĩnh (nếu cần).
- **Lưu ý**:
  - `next export` không hỗ trợ các tính năng server-side như API Routes hoặc `getServerSideProps`.
  - Sử dụng `next start` để chạy ứng dụng production sau khi build (`npm run start`).
- **Ưu điểm**:
  - Tối ưu hóa hiệu suất với bundle nhỏ gọn.
  - `next export` phù hợp cho các trang tĩnh triển khai trên CDN.
- **Nhược điểm**:
  - `next export` không hỗ trợ các tính năng động (SSR, API Routes).
  - Cần cấu hình cẩn thận để đảm bảo tất cả các trang đều tương thích với export.
- **Trường hợp sử dụng**: Triển khai các trang tĩnh (blog, landing page) hoặc ứng dụng cần tối ưu hóa hiệu suất production.

## 2. Triển khai trên Vercel (ưu tiên)

- **Mục đích**: Vercel là nền tảng được tối ưu hóa cho Next.js, cung cấp triển khai dễ dàng, tự động mở rộng, và hỗ trợ tất cả các tính năng của Next.js.
- **Cách thực hiện**:
  1. **Cài đặt Vercel CLI**:
     - `npm install -g vercel`
  2. **Đăng nhập**:
     - `vercel login`
  3. **Triển khai dự án**:
     - Chạy `vercel` trong thư mục dự án để triển khai.
     - Hoặc đẩy code lên GitHub và liên kết với Vercel qua giao diện web.
  4. **Cấu hình**:
     - Vercel tự động phát hiện dự án Next.js và thiết lập build settings.
     - Có thể tùy chỉnh trong `vercel.json` nếu cần:
```json
{
  "version": 2,
  "builds": [
    {
      "src": "next.config.js",
      "use": "@vercel/next"
    }
  ]
}
```
- **Tính năng nổi bật**:
  - Tự động mở rộng (auto-scaling).
  - Hỗ trợ SSG, SSR, ISR, và API Routes.
  - Tích hợp domain tùy chỉnh, SSL miễn phí, và CDN toàn cầu.
  - Preview deployments cho mỗi commit trên Git.
- **Ưu điểm**:
  - Tích hợp chặt chẽ với Next.js (do cùng nhà phát triển).
  - Quy trình triển khai đơn giản, không cần cấu hình phức tạp.
  - Hỗ trợ preview URL để kiểm tra trước khi deploy production.
- **Nhược điểm**:
  - Chi phí có thể cao với các ứng dụng lớn hoặc lưu lượng truy cập cao.
  - Phụ thuộc vào hệ sinh thái Vercel.
- **Trường hợp sử dụng**: Hầu hết các dự án Next.js, từ blog nhỏ đến ứng dụng thương mại điện tử lớn.

## 3. Triển khai trên các nền tảng khác (Netlify, Firebase, VPS, Docker)

### a. Netlify
- **Mục đích**: Triển khai các ứng dụng tĩnh (SSG) hoặc ứng dụng Next.js với các tính năng giới hạn (không hỗ trợ SSR natively).
- **Cách thực hiện**:
  1. Chạy `next build && next export` để tạo file tĩnh trong thư mục `out`.
  2. Kéo và thả thư mục `out` vào Netlify hoặc liên kết với GitHub.
  3. Cấu hình build command trong Netlify:
     - Build command: `next build && next export`
     - Publish directory: `out`
- **Ưu điểm**:
  - Dễ triển khai các ứng dụng tĩnh.
  - Hỗ trợ domain tùy chỉnh, SSL miễn phí, và CDN.
- **Nhược điểm**:
  - Không hỗ trợ SSR hoặc API Routes trừ khi sử dụng serverless functions.
  - Hạn chế với các ứng dụng động.
- **Trường hợp sử dụng**: Các trang tĩnh như blog hoặc landing page.

### b. Firebase
- **Mục đích**: Triển khai ứng dụng Next.js (cả SSG và SSR) bằng Firebase Hosting và Cloud Functions.
- **Cách thực hiện**:
  1. Cài đặt Firebase CLI: `npm install -g firebase-tools`
  2. Đăng nhập: `firebase login`
  3. Khởi tạo dự án: `firebase init hosting`
  4. Cấu hình Firebase Hosting:
     - Build và export file tĩnh (nếu dùng SSG).
     - Sử dụng Cloud Functions để chạy SSR hoặc API Routes:
```javascript
// functions/index.js
const functions = require('firebase-functions');
const next = require('next');

const app = next({ dev: false });
const handle = app.getRequestHandler();

exports.nextApp = functions.https.onRequest((req, res) => {
  return app.prepare().then(() => handle(req, res));
});
```
  5. Triển khai: `firebase deploy`
- **Ưu điểm**:
  - Hỗ trợ cả SSG và SSR thông qua Cloud Functions.
  - Tích hợp tốt với các dịch vụ Firebase khác (Firestore, Authentication).
- **Nhược điểm**:
  - Cần cấu hình thêm để hỗ trợ SSR.
  - Chi phí Cloud Functions có thể cao với lưu lượng lớn.
- **Trường hợp sử dụng**: Ứng dụng cần tích hợp với hệ sinh thái Firebase.

### c. VPS (Virtual Private Server)
- **Mục đích**: Triển khai ứng dụng Next.js trên máy chủ tùy chỉnh (như AWS EC2, DigitalOcean).
- **Cách thực hiện**:
  1. Cài đặt Node.js và PM2 (hoặc tương tự) trên VPS.
  2. Clone dự án từ GitHub hoặc upload mã nguồn.
  3. Chạy `npm install && npm run build`.
  4. Khởi động ứng dụng: `npm run start` hoặc sử dụng PM2: `pm2 start npm --name "next-app" -- start`.
  5. Cấu hình reverse proxy (Nginx/Apache) để xử lý yêu cầu HTTP.
- **Ví dụ Nginx config**:
```nginx
server {
  listen 80;
  server_name your-domain.com;

  location / {
    proxy_pass http://localhost:3000;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
  }
}
```
- **Ưu điểm**:
  - Toàn quyền kiểm soát server.
  - Hỗ trợ tất cả tính năng Next.js (SSR, API Routes, ISR).
- **Nhược điểm**:
  - Cần kiến thức quản trị server.
  - Không tự động mở rộng như Vercel.
- **Trường hợp sử dụng**: Ứng dụng cần tùy chỉnh server hoặc tích hợp với hệ thống nội bộ.

### d. Docker
- **Mục đích**: Đóng gói ứng dụng Next.js vào container để triển khai trên bất kỳ nền tảng nào hỗ trợ Docker.
- **Cách thực hiện**:
  1. Tạo file `Dockerfile`:
```dockerfile
# Dockerfile
FROM node:18-alpine

WORKDIR /app

COPY package*.json ./
RUN npm install

COPY . .
RUN npm run build

EXPOSE 3000
CMD ["npm", "start"]
```
  2. Tạo file `.dockerignore` để bỏ qua các file không cần thiết:
```
node_modules
.next
out
```
  3. Build và chạy container:
     - `docker build -t next-app .`
     - `docker run -p 3000:3000 next-app`
  4. Triển khai lên dịch vụ như AWS ECS, Kubernetes, hoặc Docker Swarm.
- **Ưu điểm**:
  - Đảm bảo môi trường nhất quán trên các nền tảng.
  - Dễ dàng triển khai trên nhiều dịch vụ cloud.
- **Nhược điểm**:
  - Cần kiến thức về Docker.
  - Có thể phức tạp khi tích hợp với các dịch vụ khác.
- **Trường hợp sử dụng**: Ứng dụng cần triển khai trên nhiều môi trường hoặc tích hợp với CI/CD.

## 4. Biến môi trường .env trong Next.js

- **Mục đích**: Quản lý các biến môi trường (environment variables) để lưu trữ thông tin nhạy cảm (API key, database URL, v.v.) hoặc cấu hình khác nhau giữa các môi trường (development, production).
- **Cách thực hiện**:
  - Tạo file `.env.local` cho môi trường phát triển và `.env.production` cho production.
  - Các biến có tiền tố `NEXT_PUBLIC_` sẽ được nhúng vào client-side bundle.
- **Ví dụ**:
```plaintext
# .env.local
DATABASE_URL=mongodb://localhost:27017/myapp
NEXT_PUBLIC_API_URL=http://localhost:3000/api
```
- **Sử dụng**:
  - Server-side (trong `getServerSideProps`, API Routes, v.v.):
```javascript
// pages/api/data.js
export default function handler(req, res) {
  const dbUrl = process.env.DATABASE_URL;
  res.status(200).json({ dbUrl });
}
```
  - Client-side (trong component):
```javascript
// components/ApiUrl.js
export default function ApiUrl() {
  return <div>API URL: {process.env.NEXT_PUBLIC_API_URL}</div>;
}
```
- **Cấu hình trên Vercel**:
  - Thêm biến môi trường trong dashboard Vercel (Settings > Environment Variables).
  - Vercel tự động inject các biến này vào ứng dụng khi build.
- **Lưu ý**:
  - Không lưu thông tin nhạy cảm trong các biến `NEXT_PUBLIC_` vì chúng sẽ lộ ra client-side.
  - Sử dụng `.env.development` và `.env.production` để tách biệt cấu hình.
  - Đảm bảo file `.env*` được thêm vào `.gitignore` để không đẩy lên Git.
- **Ưu điểm**:
  - Dễ dàng quản lý cấu hình theo môi trường.
  - Tích hợp tốt với Vercel và các nền tảng khác.
- **Nhược điểm**:
  - Cần cẩn thận với các biến `NEXT_PUBLIC_` để tránh lộ thông tin.
  - Quản lý nhiều file `.env` có thể phức tạp với dự án lớn.
- **Trường hợp sử dụng**: Lưu trữ API key, URL cơ sở dữ liệu, hoặc các cấu hình khác nhau giữa các môi trường.