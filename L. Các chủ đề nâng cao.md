# XI. Các chủ đề nâng cao trong Next.js

## 1. Middleware trong Next.js 13+

- **Mục đích**: Middleware cho phép chạy code trước khi xử lý yêu cầu HTTP, dùng để xác thực, chuyển hướng, hoặc chỉnh sửa request/response.
- **Cách hoạt động**:
  - Middleware chạy ở tầng edge hoặc server, được định nghĩa trong file `middleware.js` (hoặc `.ts`) ở thư mục gốc dự án.
  - Hỗ trợ xử lý các yêu cầu như xác thực, logging, hoặc chuyển hướng dựa trên logic.
- **Ví dụ**:
```javascript
// middleware.js
import { NextResponse } from 'next/server';

export function middleware(request) {
  const { pathname } = request.nextUrl;

  // Chuyển hướng nếu truy cập trang admin mà không có token
  if (pathname.startsWith('/admin') && !request.cookies.has('auth_token')) {
    return NextResponse.redirect(new URL('/login', request.url));
  }

  // Thêm header tùy chỉnh
  const response = NextResponse.next();
  response.headers.set('x-custom-header', 'my-value');
  return response;
}

export const config = {
  matcher: ['/admin/:path*', '/dashboard/:path*'], // Áp dụng middleware cho các route cụ thể
};
```
- **Ưu điểm**:
  - Chạy trước khi render trang, giúp xử lý logic sớm.
  - Hỗ trợ tại tầng edge, giảm độ trễ.
  - Tích hợp tốt với các tính năng Next.js như i18n và authentication.
- **Nhược điểm**:
  - Hạn chế về kích thước code (vì chạy ở edge, thường giới hạn 1MB).
  - Cần cấu hình `matcher` cẩn thận để tránh ảnh hưởng đến các route không mong muốn.
- **Trường hợp sử dụng**: Xác thực người dùng, chuyển hướng ngôn ngữ, hoặc thêm header tùy chỉnh.

## 2. Edge Functions vs Server Functions

- **Edge Functions**:
  - **Mục đích**: Xử lý logic tại tầng edge (CDN), gần với người dùng để giảm độ trễ.
  - **Cách hoạt động**:
    - Chạy trên môi trường edge runtime (như Vercel Edge Functions).
    - Thường dùng cho các tác vụ nhẹ như chuyển hướng, xác thực đơn giản, hoặc chỉnh sửa response.
  - **Ví dụ**:
```javascript
// pages/api/edge.js
export const config = {
  runtime: 'edge',
};

export default async function handler(req) {
  return new Response(JSON.stringify({ message: 'Chạy tại Edge' }), {
    status: 200,
    headers: { 'Content-Type': 'application/json' },
  });
}
```
  - **Ưu điểm**:
    - Độ trễ thấp do chạy gần người dùng.
    - Tự động mở rộng (serverless).
  - **Nhược điểm**:
    - Hạn chế về tính năng (không hỗ trợ đầy đủ Node.js APIs).
    - Giới hạn kích thước và thời gian chạy.

- **Server Functions**:
  - **Mục đích**: Xử lý logic phức tạp tại server Node.js, phù hợp cho các tác vụ nặng như truy vấn cơ sở dữ liệu.
  - **Cách hoạt động**:
    - Chạy trong môi trường Node.js (mặc định cho API Routes).
    - Hỗ trợ đầy đủ các API của Node.js.
  - **Ví dụ**:
```javascript
// pages/api/server.js
export default async function handler(req, res) {
  const data = await fetchDataFromDatabase(); // Giả sử truy vấn DB
  res.status(200).json({ data });
}
```
  - **Ưu điểm**:
    - Hỗ trợ đầy đủ các thư viện Node.js.
    - Phù hợp cho logic phức tạp hoặc tác vụ nặng.
  - **Nhược điểm**:
    - Độ trễ cao hơn so với Edge Functions.
    - Không tự động mở rộng tốt như edge.

- **So sánh**:
  - Edge Functions: Nhanh, nhẹ, gần người dùng, nhưng hạn chế về tính năng.
  - Server Functions: Linh hoạt, mạnh mẽ, nhưng có độ trễ cao hơn.
- **Trường hợp sử dụng**:
  - Edge Functions: Xác thực, chuyển hướng, hoặc xử lý yêu cầu đơn giản.
  - Server Functions: Truy vấn cơ sở dữ liệu, xử lý file, hoặc tích hợp với API bên ngoài.

## 3. Image Optimization nâng cao

- **Mục đích**: Tối ưu hóa hình ảnh để giảm thời gian tải, cải thiện hiệu suất và trải nghiệm người dùng.
- **Cách hoạt động**:
  - Sử dụng component `next/image` với các tính năng nâng cao như placeholder, quality, hoặc loader tùy chỉnh.
  - Hỗ trợ các định dạng hiện đại (WebP, AVIF) và tự động resize hình ảnh.
- **Ví dụ**:
```javascript
// components/AdvancedImage.js
import Image from 'next/image';

export default function AdvancedImage() {
  return (
    <Image
      src="/images/example.jpg"
      alt="Hình ảnh tối ưu"
      width={800}
      height={600}
      quality={75} // Giảm chất lượng để tối ưu kích thước
      placeholder="blur" // Hiển thị placeholder mờ khi tải
      blurDataURL="/images/placeholder.jpg" // Hình ảnh placeholder tùy chỉnh
      sizes="(max-width: 768px) 100vw, 50vw" // Tối ưu kích thước cho các màn hình
    />
  );
}
```
- **Loader tùy chỉnh**:
  - Sử dụng loader để tải hình ảnh từ nguồn bên ngoài (như CDN):
```javascript
// next.config.js
module.exports = {
  images: {
    remotePatterns: [
      {
        protocol: 'https',
        hostname: 'example.com',
      },
    ],
  },
};
```
```javascript
// components/CustomImage.js
import Image from 'next/image';

const myLoader = ({ src, width, quality }) => {
  return `https://example.com/${src}?w=${width}&q=${quality || 75}`;
};

export default function CustomImage() {
  return (
    <Image
      loader={myLoader}
      src="example.jpg"
      alt="Hình ảnh từ CDN"
      width={800}
      height={600}
    />
  );
}
```
- **Ưu điểm**:
  - Tự động tối ưu hóa hình ảnh (resize, WebP/AVIF, lazy loading).
  - Hỗ trợ placeholder và sizes để cải thiện trải nghiệm người dùng.
  - Dễ tích hợp với CDN bên ngoài.
- **Nhược điểm**:
  - Cần cấu hình thêm cho hình ảnh từ nguồn bên ngoài.
  - Có thể phức tạp khi tùy chỉnh loader.
- **Trường hợp sử dụng**: Trang web có nhiều hình ảnh, như thương mại điện tử, blog, hoặc portfolio.

## 4. Rewrite, Redirect và Header cấu hình trong next.config.js

- **Mục đích**: Tùy chỉnh cách xử lý URL, chuyển hướng, hoặc thêm header trong Next.js.
- **Cách thực hiện**:

### a. Rewrites
- Chuyển hướng nội bộ URL mà không thay đổi URL hiển thị trên trình duyệt.
- Ví dụ:
```javascript
// next.config.js
module.exports = {
  async rewrites() {
    return [
      {
        source: '/old-path/:id',
        destination: '/new-path/:id', // Chuyển hướng nội bộ
      },
      {
        source: '/api/external',
        destination: 'https://api.example.com/data', // Proxy đến API bên ngoài
      },
    ];
  },
};
```

### b. Redirects
- Chuyển hướng người dùng đến URL mới, thay đổi URL trên trình duyệt.
- Ví dụ:
```javascript
// next.config.js
module.exports = {
  async redirects() {
    return [
      {
        source: '/legacy',
        destination: '/new-page',
        permanent: true, // 301 redirect (vĩnh viễn)
      },
      {
        source: '/temp',
        destination: '/new-temp',
        permanent: false, // 302 redirect (tạm thời)
      },
    ];
  },
};
```

### c. Headers
- Thêm header tùy chỉnh cho các route.
- Ví dụ:
```javascript
// next.config.js
module.exports = {
  async headers() {
    return [
      {
        source: '/:path*',
        headers: [
          {
            key: 'X-Frame-Options',
            value: 'DENY',
          },
          {
            key: 'Cache-Control',
            value: 'public, max-age=31536000, immutable',
          },
        ],
      },
    ];
  },
};
```
- **Ưu điểm**:
  - Dễ dàng quản lý URL và header trong `next.config.js`.
  - Hỗ trợ proxy API bên ngoài mà không cần server riêng.
  - Tăng bảo mật và hiệu suất với header tùy chỉnh.
- **Nhược điểm**:
  - Cần cẩn thận khi cấu hình để tránh xung đột route.
  - Rewrites có thể phức tạp với các ứng dụng lớn.
- **Trường hợp sử dụng**: Chuyển hướng từ URL cũ, proxy API, hoặc thêm header bảo mật (như CSP, X-Frame-Options).

## 5. Phân biệt giữa App Router vs Pages Router (Next.js 13+)

- **App Router** (Next.js 13+):
  - **Mục đích**: Cấu trúc mới trong Next.js 13, sử dụng thư mục `app/` để quản lý route, layout, và các tính năng nâng cao như Server Components.
  - **Đặc điểm**:
    - Hỗ trợ React Server Components (RSC) mặc định.
    - Cấu trúc route dựa trên thư mục (file-system routing).
    - Tích hợp layout động, loading UI, và error handling.
    - Hỗ trợ Server Actions và các tính năng mới.
  - **Ví dụ**:
```plaintext
app/
  layout.js        // Layout chung
  page.js          // Trang chính (/)
  about/
    page.js        // Trang /about
  [id]/
    page.js        // Trang động /:id
```
  - **Ưu điểm**:
    - Hiệu suất tốt hơn với Server Components.
    - Quản lý layout và error UI dễ dàng hơn.
    - Tích hợp tốt với các tính năng mới (Server Actions, Metadata API).
  - **Nhược điểm**:
    - Yêu cầu học cách sử dụng mới.
    - Một số thư viện chưa tương thích hoàn toàn với Server Components.

- **Pages Router**:
  - **Mục đích**: Cấu trúc truyền thống của Next.js, sử dụng thư mục `pages/` để định nghĩa route.
  - **Đặc điểm**:
    - Sử dụng `getStaticProps`, `getServerSideProps`, và `getStaticPaths` để fetch dữ liệu.
    - Không hỗ trợ Server Components mặc định.
    - Dễ sử dụng cho các dự án đơn giản hoặc đã tồn tại.
  - **Ví dụ**:
```plaintext
pages/
  index.js         // Trang chính (/)
  about.js         // Trang /about
  [id].js          // Trang động /:id
  api/
    hello.js       // API route /api/hello
```
  - **Ưu điểm**:
    - Dễ học và sử dụng cho các dự án cũ.
    - Tương thích tốt với hầu hết các thư viện.
  - **Nhược điểm**:
    - Ít hỗ trợ các tính năng mới như Server Components.
    - Layout và error handling cần cấu hình thủ công nhiều hơn.

- **So sánh**:
  - App Router: Tương lai của Next.js, tối ưu cho hiệu suất và các tính năng mới.
  - Pages Router: Truyền thống, phù hợp cho dự án hiện tại hoặc đơn giản.
- **Trường hợp sử dụng**:
  - App Router: Dự án mới, cần tận dụng Server Components hoặc Server Actions.
  - Pages Router: Dự án cũ hoặc cần tương thích với các thư viện chưa hỗ trợ App Router.

## 6. Server Actions (Next.js 14+)

- **Mục đích**: Cho phép thực thi logic phía server trực tiếp từ client-side form hoặc hàm, thay thế cho API Routes trong một số trường hợp.
- **Cách hoạt động**:
  - Sử dụng directive `"use server"` trong file hoặc hàm.
  - Gọi Server Actions từ client thông qua form hoặc JavaScript.
- **Ví dụ**:
```javascript
// app/actions.js
'use server';

export async function createPost(formData) {
  const title = formData.get('title');
  // Giả sử lưu vào cơ sở dữ liệu
  await db.posts.create({ title });
  return { success: true };
}
```
```javascript
// app/page.js
import { createPost } from './actions';

export default function CreatePostForm() {
  return (
    <form action={createPost}>
      <input type="text" name="title" placeholder="Tiêu đề bài viết" />
      <button type="submit">Tạo bài viết</button>
    </form>
  );
}
```
- **Ưu điểm**:
  - Giảm nhu cầu tạo API Routes riêng.
  - Tích hợp chặt chẽ với form và Server Components.
  - Hỗ trợ xử lý lỗi và validation dễ dàng.
- **Nhược điểm**:
  - Chỉ hoạt động trong App Router.
  - Cần đảm bảo bảo mật (ví dụ: kiểm tra quyền truy cập).
- **Trường hợp sử dụng**: Xử lý form, cập nhật dữ liệu, hoặc thực hiện các tác vụ server-side trực tiếp từ client.

## 7. Metadata API (App Router)

- **Mục đích**: Quản lý metadata (meta tags, Open Graph, favicon, v.v.) trong App Router để tối ưu SEO và trải nghiệm người dùng.
- **Cách hoạt động**:
  - Sử dụng `metadata` object hoặc hàm `generateMetadata` trong `layout.js` hoặc `page.js`.
  - Hỗ trợ metadata động dựa trên dữ liệu từ server.
- **Ví dụ tĩnh**:
```javascript
// app/layout.js
export const metadata = {
  title: 'My Next.js App',
  description: 'Ứng dụng Next.js với đa ngôn ngữ',
  openGraph: {
    title: 'My Next.js App',
    description: 'Ứng dụng Next.js tuyệt vời',
    images: ['/images/og-image.jpg'],
  },
};

export default function RootLayout({ children }) {
  return (
    <html lang="vi">
      <body>{children}</body>
    </html>
  );
}
```
- **Ví dụ động**:
```javascript
// app/[id]/page.js
export async function generateMetadata({ params }) {
  const id = params.id;
  const post = await fetchPost(id); // Giả sử lấy dữ liệu bài viết

  return {
    title: post.title,
    description: post.excerpt,
    openGraph: {
      title: post.title,
      description: post.excerpt,
      images: [post.image],
    },
  };
}

export default function PostPage({ params }) {
  return <h1>Bài viết {params.id}</h1>;
}
```
- **Ưu điểm**:
  - Tích hợp dễ dàng trong App Router.
  - Hỗ trợ metadata động dựa trên dữ liệu server.
  - Tốt cho SEO và chia sẻ mạng xã hội.
- **Nhược điểm**:
  - Chỉ hoạt động trong App Router.
  - Cần quản lý metadata cẩn thận để tránh trùng lặp.
- **Trường hợp sử dụng**: Tối ưu SEO cho các trang động, như bài viết blog hoặc sản phẩm thương mại điện tử.