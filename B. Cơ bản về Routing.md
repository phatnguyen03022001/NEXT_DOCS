# II. Cơ bản về Routing

## 1. File-based Routing trong Next.js

Next.js sử dụng hệ thống **File-based Routing**, trong đó cấu trúc thư mục trong `app/` (Next.js 13+) hoặc `pages/` (Next.js 12 trở về trước) xác định các tuyến đường (routes) của ứng dụng.

- **Cách hoạt động**:
  - Mỗi file `page.js` (hoặc `index.js` trong `pages/`) trong thư mục `app/` tương ứng với một route.
  - Ví dụ:
    - `app/page.js` → Route `/`
    - `app/about/page.js` → Route `/about`
    - `app/blog/page.js` → Route `/blog`

- **Lưu ý**:
  - File phải xuất một React component làm mặc định (`export default`).
  - Next.js tự động xử lý code splitting cho mỗi trang, tối ưu hiệu suất.
  - Các file không phải `page.js` (VD: components, utils) không ảnh hưởng đến routing.

**Ví dụ**:
```jsx
// app/about/page.js
export default function About() {
  return <h1>Giới thiệu</h1>;
}
```
→ Truy cập: `http://localhost:3000/about`

---

## 2. Dynamic Routing (Routes động)

Next.js hỗ trợ các route động bằng cách sử dụng cú pháp `[param]` trong tên thư mục hoặc file.

- **Cách tạo**:
  - Tạo thư mục hoặc file với tên `[param]` trong `app/`.
  - `param` là tham số động, có thể truy xuất thông qua `useParams` (App Router) hoặc `getServerSideProps`/`getStaticProps` (Pages Router).

- **Ví dụ**:
  - Thư mục: `app/blog/[id]/page.js`
  - Route: `/blog/123` → `id = 123`

```jsx
// app/blog/[id]/page.js
import { useParams } from 'next/navigation';

export default function BlogPost() {
  const params = useParams();
  return <h1>Blog ID: {params.id}</h1>;
}
```

- **Truy cập**:
  - `/blog/1` → Hiển thị "Blog ID: 1"
  - `/blog/abc` → Hiển thị "Blog ID: abc"

---

## 3. Catch-all và Optional Catch-all Routes

### Catch-all Routes
- Dùng để xử lý nhiều đoạn đường dẫn (path segments) trong một route.
- Cú pháp: `[...param]` (thư mục hoặc file).
- Ví dụ: `app/shop/[...slug]/page.js`
  - Route: `/shop/a/b/c` → `slug = ['a', 'b', 'c']`

```jsx
// app/shop/[...slug]/page.js
import { useParams } from 'next/navigation';

export default function Shop() {
  const params = useParams();
  return <h1>Shop Segments: {params.slug.join('/')}</h1>;
}
```

- **Truy cập**:
  - `/shop/electronics/phones/iphone` → Hiển thị "Shop Segments: electronics/phones/iphone"

### Optional Catch-all Routes
- Tương tự Catch-all, nhưng cho phép route gốc (không có tham số) hợp lệ.
- Cú pháp: `[[...param]]`
- Ví dụ: `app/product/[[...slug]]/page.js`
  - Route: `/product` (hợp lệ), `/product/a/b` → `slug = ['a', 'b']`

```jsx
// app/product/[[...slug]]/page.js
import { useParams } from 'next/navigation';

export default function Product() {
  const params = useParams();
  const slug = params.slug || [];
  return <h1>Product Segments: {slug.length ? slug.join('/') : 'No segments'}</h1>;
}
```

---

## 4. Nested Routing (Lồng Routes)

Next.js hỗ trợ các route lồng nhau bằng cách tạo các thư mục con trong `app/`.

- **Cách hoạt động**:
  - Thư mục con trong `app/` tạo ra các route con.
  - Ví dụ:
    - `app/dashboard/settings/page.js` → Route `/dashboard/settings`
    - `app/dashboard/profile/page.js` → Route `/dashboard/profile`

- **Sử dụng Layout**:
  - File `layout.js` trong thư mục cha (VD: `app/dashboard/layout.js`) áp dụng cho tất cả các route con.
  - Layout lồng nhau được hỗ trợ, cho phép tái sử dụng giao diện.

```jsx
// app/dashboard/layout.js
export default function DashboardLayout({ children }) {
  return (
    <div>
      <h1>Dashboard Header</h1>
      {children}
    </div>
  );
}

// app/dashboard/settings/page.js
export default function Settings() {
  return <h2>Settings Page</h2>;
}
```

- **Kết quả**:
  - Truy cập `/dashboard/settings` → Hiển thị "Dashboard Header" và "Settings Page" trong layout.

---

## 5. Link Component và Navigation giữa các Trang

Next.js cung cấp component `<Link>` để điều hướng giữa các trang mà không cần tải lại toàn bộ trang, tối ưu hiệu suất.

- **Cài đặt**:
  - `<Link>` thuộc module `next/link`, không cần cài đặt thêm.
  - Hỗ trợ prefetching (tải trước trang trong nền) để tăng tốc độ chuyển trang.

- **Ví dụ sử dụng**:
```jsx
// app/page.js
import Link from 'next/link';

export default function Home() {
  return (
    <div>
      <h1>Trang chủ</h1>
      <Link href="/about">Đi đến trang Giới thiệu</Link>
      <Link href="/blog/123">Đi đến Blog 123</Link>
    </div>
  );
}
```

- **Thuộc tính quan trọng**:
  - `href`: Đường dẫn đến trang đích (bắt buộc).
  - `replace`: Thay thế lịch sử trình duyệt thay vì thêm mới (`replace={true}`).
  - `prefetch`: Tắt prefetching nếu không cần (`prefetch={false}`).

- **Điều hướng lập trình**:
  - Sử dụng `useRouter` (App Router) để điều hướng bằng code.
  ```jsx
  import { useRouter } from 'next/navigation';

  export default function MyComponent() {
    const router = useRouter();

    const handleClick = () => {
      router.push('/about');
    };

    return <button onClick={handleClick}>Đi đến Giới thiệu</button>;
  }
  ```

- **Lưu ý**:
  - `<Link>` chỉ hoạt động với các route nội bộ. Với link bên ngoài, sử dụng thẻ `<a>` thông thường.
  - Prefetching tự động bật trong môi trường production, nhưng có thể tắt để tối ưu tài nguyên.