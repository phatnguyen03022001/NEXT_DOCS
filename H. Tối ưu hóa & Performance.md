# VIII. Tối ưu hóa & Performance trong Next.js

## 1. Tối ưu SEO với Next.js (meta tags, Open Graph, Sitemap, Robots.txt)

- **Mục đích**: Tăng khả năng hiển thị trên công cụ tìm kiếm (SEO) và cải thiện trải nghiệm chia sẻ trên mạng xã hội.
- **Cách thực hiện**:

### a. Meta Tags
- Sử dụng component `<Head>` từ `next/head` để thêm các thẻ meta vào HTML.
- Ví dụ:
```javascript
// pages/index.js
import Head from 'next/head';

export default function Home() {
  return (
    <>
      <Head>
        <title>Trang chủ - My Next.js App</title>
        <meta name="description" content="Đây là trang chủ của ứng dụng Next.js" />
        <meta name="keywords" content="Next.js, SEO, React" />
        <meta name="viewport" content="width=device-width, initial-scale=1.0" />
      </Head>
      <h1>Chào mừng đến với Next.js</h1>
    </>
  );
}
```

### b. Open Graph (OG) Tags
- Thêm thẻ Open Graph để tối ưu hóa chia sẻ trên mạng xã hội (Facebook, Twitter, v.v.).
- Ví dụ:
```javascript
// pages/index.js
import Head from 'next/head';

export default function Home() {
  return (
    <>
      <Head>
        <title>Trang chủ - My Next.js App</title>
        <meta name="description" content="Đây là trang chủ của ứng dụng Next.js" />
        <meta property="og:title" content="Trang chủ - My Next.js App" />
        <meta property="og:description" content="Khám phá ứng dụng Next.js tuyệt vời" />
        <meta property="og:image" content="/images/preview.jpg" />
        <meta property="og:url" content="https://your-site.com" />
        <meta property="og:type" content="website" />
      </Head>
      <h1>Chào mừng đến với Next.js</h1>
    </>
  );
}
```

### c. Sitemap
- Tạo file `sitemap.xml` để liệt kê các URL của trang web, giúp công cụ tìm kiếm dễ dàng thu thập dữ liệu.
- Cách tạo tự động với `next-sitemap`:
  - Cài đặt: `npm install next-sitemap`
  - Tạo file cấu hình `next-sitemap.config.js`:
```javascript
/** @type {import('next-sitemap').IConfig} */
module.exports = {
  siteUrl: 'https://your-site.com',
  generateRobotsTxt: true, // Tạo file robots.txt
  sitemapSize: 7000,
};
```
  - Thêm script vào `package.json`:
```json
"scripts": {
  "postbuild": "next-sitemap"
}
```
  - Chạy `npm run build` để tạo `sitemap.xml` trong thư mục `public`.

### d. Robots.txt
- Điều khiển cách công cụ tìm kiếm thu thập dữ liệu trang web.
- Tạo file `public/robots.txt`:
```
User-agent: *
Allow: /
Disallow: /admin/
Sitemap: https://your-site.com/sitemap.xml
```
- Hoặc sử dụng `next-sitemap` để tự động tạo `robots.txt` như trên.
- **Ưu điểm**:
  - Cải thiện thứ hạng SEO với meta tags và sitemap.
  - Tăng trải nghiệm chia sẻ trên mạng xã hội với Open Graph.
  - Dễ dàng cấu hình với các công cụ tích hợp sẵn.
- **Nhược điểm**:
  - Cần quản lý nội dung meta cho từng trang.
  - Sitemap động có thể phức tạp với các trang lớn.
- **Trường hợp sử dụng**: Blog, trang thương mại điện tử, hoặc bất kỳ trang web nào cần tối ưu hóa SEO.

## 2. Phân trang (Pagination)

- **Mục đích**: Chia dữ liệu thành các trang để cải thiện hiệu suất và trải nghiệm người dùng khi tải danh sách lớn.
- **Cách thực hiện**:
  - Sử dụng `getStaticProps` hoặc `getServerSideProps` để lấy dữ liệu phân trang.
  - Tạo các route động (ví dụ: `/posts/page/[page]`) hoặc query params (`/posts?page=1`).
- **Ví dụ với getStaticProps**:
```javascript
// pages/posts/page/[page].js
import Link from 'next/link';

export default function Posts({ posts, currentPage, totalPages }) {
  return (
    <div>
      <h1>Danh sách bài viết - Trang {currentPage}</h1>
      <ul>
        {posts.map((post) => (
          <li key={post.id}>{post.title}</li>
        ))}
      </ul>
      <div>
        {currentPage > 1 && (
          <Link href={`/posts/page/${currentPage - 1}`}>Trang trước</Link>
        )}
        {currentPage < totalPages && (
          <Link href={`/posts/page/${currentPage + 1}`}>Trang sau</Link>
        )}
      </div>
    </div>
  );
}

export async function getStaticProps({ params }) {
  const page = parseInt(params.page);
  const perPage = 10; // Số bài viết mỗi trang
  const res = await fetch(`https://api.example.com/posts?_page=${page}&_limit=${perPage}`);
  const posts = await res.json();
  const totalPosts = parseInt(res.headers.get('x-total-count')); // Tổng số bài viết
  const totalPages = Math.ceil(totalPosts / perPage);

  return {
    props: {
      posts,
      currentPage: page,
      totalPages,
    },
  };
}

export async function getStaticPaths() {
  const perPage = 10;
  const res = await fetch('https://api.example.com/posts');
  const posts = await res.json();
  const totalPages = Math.ceil(posts.length / perPage);

  const paths = Array.from({ length: totalPages }, (_, i) => ({
    params: { page: `${i + 1}` },
  }));

  return {
    paths,
    fallback: false,
  };
}
```
- **Ưu điểm**:
  - Cải thiện hiệu suất bằng cách chỉ tải một phần dữ liệu.
  - Tốt cho SEO khi sử dụng SSG với các trang tĩnh.
  - Cải thiện trải nghiệm người dùng với giao diện phân trang.
- **Nhược điểm**:
  - Cần xử lý logic phân trang ở cả client và server.
  - Có thể phức tạp với dữ liệu động lớn.
- **Trường hợp sử dụng**: Danh sách bài viết, sản phẩm, hoặc bất kỳ nội dung nào cần hiển thị theo từng trang.

## 3. Tối ưu hình ảnh với next/image

- **Mục đích**: Tối ưu hóa hình ảnh để giảm thời gian tải và cải thiện hiệu suất trang web.
- **Cách hoạt động**:
  - Component `next/image` tự động tối ưu hóa hình ảnh (nén, resize, lazy loading).
  - Hỗ trợ các định dạng hiện đại như WebP.
  - Tự động tạo nhiều kích thước hình ảnh cho các thiết bị khác nhau.
- **Ví dụ**:
```javascript
// components/MyImage.js
import Image from 'next/image';

export default function MyImage() {
  return (
    <Image
      src="/images/example.jpg"
      alt="Hình ảnh ví dụ"
      width={500}
      height={300}
      layout="responsive" // Hoặc "fixed", "intrinsic", "fill"
      priority // Ưu tiên tải trước cho hình ảnh quan trọng
    />
  );
}
```
- **Cấu hình với domain bên ngoài**:
  - Thêm domain vào `next.config.js` để cho phép tải hình ảnh từ nguồn bên ngoài:
```javascript
/** @type {import('next').NextConfig} */
module.exports = {
  images: {
    domains: ['example.com'],
  },
};
```
- **Ưu điểm**:
  - Tự động tối ưu hóa hình ảnh (nén, resize, WebP).
  - Hỗ trợ lazy loading, giảm thời gian tải ban đầu.
  - Phù hợp với các thiết bị có độ phân giải khác nhau.
- **Nhược điểm**:
  - Cần cấu hình domain cho hình ảnh bên ngoài.
  - Yêu cầu chỉ định kích thước (width/height) hoặc layout.
- **Trường hợp sử dụng**: Trang web có nhiều hình ảnh, như blog, thương mại điện tử, hoặc portfolio.

## 4. Code splitting & lazy loading

- **Mục đích**: Giảm kích thước bundle JavaScript được tải ban đầu, cải thiện thời gian tải trang.
- **Code Splitting**:
  - Next.js tự động chia nhỏ code theo trang (mỗi trang là một bundle riêng).
  - Sử dụng dynamic imports để tải các component chỉ khi cần.
- **Lazy Loading**:
  - Sử dụng `next/dynamic` để tải component động, giảm tải ban đầu.
- **Ví dụ**:
```javascript
// pages/index.js
import dynamic from 'next/dynamic';

// Tải component chỉ khi cần
const HeavyComponent = dynamic(() => import('../components/HeavyComponent'), {
  loading: () => <p>Đang tải...</p>, // Hiển thị trong lúc tải
  ssr: false, // Tắt SSR nếu chỉ cần tải phía client
});

export default function Home() {
  return (
    <div>
      <h1>Trang chủ</h1>
      <HeavyComponent />
    </div>
  );
}
```
- **Ưu điểm**:
  - Giảm kích thước bundle ban đầu, cải thiện tốc độ tải.
  - Tải component theo nhu cầu, giảm tải server và client.
  - Tích hợp dễ dàng với Next.js.
- **Nhược điểm**:
  - Có thể gây trễ nhẹ khi tải component động.
  - Cần quản lý trạng thái loading để tránh trải nghiệm xấu.
- **Trường hợp sử dụng**: Các component nặng (chart, map, hoặc thư viện lớn) hoặc trang có nội dung không cần tải ngay.

## 5. Prefetching và performance tips

- **Prefetching**:
  - Next.js tự động prefetch các trang liên kết bằng `<Link>` trong viewport khi ở chế độ production.
  - Điều này giúp chuyển đổi trang nhanh hơn vì dữ liệu đã được tải trước.
- **Ví dụ**:
```javascript
// pages/index.js
import Link from 'next/link';

export default function Home() {
  return (
    <div>
      <h1>Trang chủ</h1>
      <Link href="/about">Về chúng tôi</Link> {/* Prefetch tự động */}
    </div>
  );
}
```
- **Tắt prefetch nếu không cần**:
```javascript
<Link href="/about" prefetch={false}>Về chúng tôi</Link>
```

### Performance Tips
1. **Sử dụng SSG/ISR**: Tận dụng Static Site Generation hoặc Incremental Static Regeneration để giảm tải server và tăng tốc độ tải.
2. **Tối ưu hóa font**:
   - Sử dụng `next/font` để tải font tối ưu:
```javascript
// pages/_app.js
import { Inter } from 'next/font/google';

const inter = Inter({ subsets: ['latin'] });

export default function MyApp({ Component, pageProps }) {
  return (
    <main className={inter.className}>
      <Component {...pageProps} />
    </main>
  );
}
```
3. **Giảm JavaScript không cần thiết**:
   - Sử dụng `next/dynamic` để lazy load thư viện lớn.
   - Loại bỏ các thư viện không cần thiết hoặc thay bằng giải pháp nhẹ hơn.
4. **Tối ưu API Routes**:
   - Sử dụng caching trong API Routes để giảm tải server:
```javascript
// pages/api/data.js
export default function handler(req, res) {
  res.setHeader('Cache-Control', 's-maxage=60, stale-while-revalidate');
  res.json({ data: 'Dữ liệu được cache' });
}
```
5. **Phân tích hiệu suất**:
   - Sử dụng `next build` để kiểm tra kích thước bundle.
   - Dùng công cụ như Lighthouse để phân tích hiệu suất và SEO.
6. **Sử dụng CDN**: Triển khai ứng dụng trên Vercel hoặc CDN khác để tăng tốc độ tải tài nguyên tĩnh.
- **Ưu điểm**:
  - Prefetching cải thiện tốc độ chuyển đổi trang.
  - Các mẹo tối ưu hóa giúp giảm thời gian tải và cải thiện trải nghiệm người dùng.
- **Nhược điểm**:
  - Cần cấu hình cẩn thận để tránh tải trước quá nhiều tài nguyên.
  - Một số tối ưu hóa có thể làm tăng độ phức tạp của code.
- **Trường hợp sử dụng**: Ứng dụng cần hiệu suất cao, như trang thương mại điện tử, blog, hoặc ứng dụng có lượng truy cập lớn.