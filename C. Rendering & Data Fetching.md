# I. Rendering & Data Fetching trong Next.js

## 1. CSR vs SSR vs SSG vs ISR

### a. Client-Side Rendering (CSR) - Kết xuất phía máy khách
- **Định nghĩa**: Kết xuất được thực hiện trên trình duyệt bằng JavaScript. Máy chủ gửi một file HTML tối thiểu, và máy khách sẽ lấy dữ liệu và kết xuất nội dung một cách động.
- **Ưu điểm**:
  - Tải trang ban đầu nhanh (HTML tối thiểu).
  - Phù hợp cho nội dung động, cụ thể theo người dùng.
  - Giảm tải cho máy chủ vì kết xuất diễn ra phía máy khách.
- **Nhược điểm**:
  - SEO kém nếu không có cấu hình bổ sung (ví dụ: pre-rendering).
  - Thời gian hiển thị nội dung ban đầu chậm hơn do dữ liệu được lấy sau khi trang tải.
  - Phụ thuộc nhiều vào JavaScript phía máy khách.
- **Trường hợp sử dụng**: Các ứng dụng một trang (SPA) với dữ liệu động, như bảng điều khiển (dashboard).

### b. Server-Side Rendering (SSR) - Kết xuất phía máy chủ
- **Định nghĩa**: Máy chủ tạo toàn bộ HTML cho trang mỗi khi nhận được yêu cầu, lấy dữ liệu trước khi gửi phản hồi tới máy khách.
- **Ưu điểm**:
  - Tốt cho SEO vì nội dung được kết xuất đầy đủ trước khi gửi đến máy khách.
  - Hiển thị ban đầu nhanh hơn (nội dung có sẵn ngay lập tức).
  - Lý tưởng cho nội dung động thay đổi thường xuyên.
- **Nhược điểm**:
  - Tải trang chậm hơn do máy chủ xử lý mỗi yêu cầu.
  - Tăng tải cho máy chủ so với các phương pháp tĩnh.
- **Trường hợp sử dụng**: Các trang có dữ liệu cập nhật thường xuyên hoặc cụ thể theo người dùng, như trang sản phẩm thương mại điện tử.

### c. Static Site Generation (SSG) - Tạo trang tĩnh
- **Định nghĩa**: HTML được tạo tại thời điểm xây dựng (build time) và được phục vụ dưới dạng file tĩnh.
- **Ưu điểm**:
  - Tải trang cực nhanh (file tĩnh được phục vụ qua CDN).
  - Tốt cho SEO vì nội dung được kết xuất sẵn.
  - Giảm tải máy chủ vì các trang được xây dựng sẵn.
- **Nhược điểm**:
  - Không phù hợp cho nội dung động cao trừ khi kết hợp với lấy dữ liệu phía máy khách hoặc ISR.
  - Thời gian xây dựng lâu cho các trang web lớn với nhiều trang.
- **Trường hợp sử dụng**: Blog, trang tài liệu, hoặc trang tiếp thị có nội dung chủ yếu là tĩnh.

### d. Incremental Static Regeneration (ISR) - Tái tạo tĩnh tăng dần
- **Định nghĩa**: Kết hợp giữa SSG và SSR. Các trang được tạo tại thời điểm xây dựng nhưng có thể được tái tạo nền sau một khoảng thời gian định sẵn (revalidate) mà không cần xây dựng lại toàn bộ.
- **Ưu điểm**:
  - Kết hợp lợi ích của SSG (tải nhanh, tốt cho SEO) với khả năng cập nhật động.
  - Không cần xây dựng lại toàn bộ trang web khi có cập nhật.
  - Phù hợp cho các trang web lớn.
- **Nhược điểm**:
  - Nội dung có thể hơi cũ cho đến khi được tái tạo.
  - Cần cấu hình cẩn thận khoảng thời gian tái tạo.
- **Trường hợp sử dụng**: Các trang web có nội dung cập nhật không quá thường xuyên, như trang tin tức hoặc danh sách sản phẩm.

## 2. getStaticProps – Tạo trang tĩnh
- **Mục đích**: Lấy dữ liệu tại thời điểm xây dựng cho Tạo trang tĩnh (SSG).
- **Cách hoạt động**: Chạy trong quá trình xây dựng, dữ liệu lấy được dùng để tạo HTML tĩnh.
- **Đặc điểm chính**:
  - Trả về một object `props` để truyền dữ liệu cho component trang.
  - Có thể kết hợp với `getStaticPaths` cho các route động.
  - Không chạy phía máy khách, nên không truy cập trực tiếp vào API trình duyệt.
- **Ví dụ**:
```javascript
export async function getStaticProps(context) {
  const res = await fetch('https://api.example.com/data');
  const data = await res.json();

  return {
    props: {
      data, // Truyền vào component trang dưới dạng props
    },
  };
}
```
- **Trường hợp sử dụng**: Kết xuất trước các bài viết blog, trang sản phẩm, hoặc nội dung ít thay đổi.

## 3. getServerSideProps – Kết xuất phía máy chủ
- **Mục đích**: Lấy dữ liệu cho mỗi yêu cầu để thực hiện Kết xuất phía máy chủ (SSR).
- **Cách hoạt động**: Chạy trên máy chủ cho mỗi yêu cầu, tạo HTML mới với dữ liệu mới nhất.
- **Đặc điểm chính**:
  - Có quyền truy cập vào object yêu cầu (`context`), hỗ trợ lấy dữ liệu cụ thể theo người dùng (ví dụ: cookie, query params).
  - Chậm hơn SSG nhưng đảm bảo nội dung luôn cập nhật.
- **Ví dụ**:
```javascript
export async function getServerSideProps(context) {
  const { params, req } = context;
  const res = await fetch(`https://api.example.com/data/${params.id}`);
  const data = await res.json();

  return {
    props: {
      data, // Truyền vào component trang dưới dạng props
    },
  };
}
```
- **Trường hợp sử dụng**: Các trang cần dữ liệu thời gian thực, như hồ sơ người dùng hoặc bảng điều khiển trực tiếp.

## 4. getStaticPaths – Tạo trang tĩnh động
- **Mục đích**: Xác định các route động cho Tạo trang tĩnh khi sử dụng các route động (ví dụ: `[id].js`).
- **Cách hoạt động**: Chỉ định các đường dẫn cần được kết xuất trước tại thời điểm xây dựng và hành vi dự phòng (fallback) cho các đường dẫn không xác định.
- **Đặc điểm chính**:
  - Trả về một object chứa danh sách các đường dẫn (`paths`) và `fallback` (có thể là `true`, `false`, hoặc `'blocking'`).
  - `fallback: false`: 404 cho các đường dẫn không được tạo trước.
  - `fallback: true` hoặc `'blocking'`: Cho phép tạo trang động khi yêu cầu.
- **Ví dụ**:
```javascript
export async function getStaticPaths() {
  const res = await fetch('https://api.example.com/posts');
  const posts = await res.json();

  const paths = posts.map((post) => ({
    params: { id: post.id.toString() },
  }));

  return {
    paths,
    fallback: false, // Hoặc true/'blocking' tùy thuộc yêu cầu
  };
}
```
- **Trường hợp sử dụng**: Các trang động như bài viết blog hoặc sản phẩm dựa trên ID.

## 5. Incremental Static Regeneration (ISR)
- **Mục đích**: Cho phép cập nhật nội dung tĩnh mà không cần xây dựng lại toàn bộ trang web.
- **Cách hoạt động**: Trong `getStaticProps`, thêm thuộc tính `revalidate` để chỉ định khoảng thời gian (tính bằng giây) sau đó trang sẽ được tái tạo.
- **Ví dụ**:
```javascript
export async function getStaticProps(context) {
  const res = await fetch('https://api.example.com/data');
  const data = await res.json();

  return {
    props: {
      data,
    },
    revalidate: 10, // Tái tạo sau 10 giây nếu có yêu cầu
  };
}
```
- **Trường hợp sử dụng**: Trang tin tức, danh sách sản phẩm, hoặc bất kỳ nội dung nào cần cập nhật định kỳ.

## 6. Client-side Data Fetching – Lấy dữ liệu phía máy khách (SWR hoặc React Query)
- **Mục đích**: Lấy dữ liệu động phía máy khách sau khi trang được tải, thường sử dụng các thư viện như **SWR** hoặc **React Query**.
- **Cách hoạt động**: Sử dụng các hook để lấy và quản lý dữ liệu, hỗ trợ caching, refetching, và xử lý trạng thái tải.
- **SWR**:
  - Thư viện nhẹ từ Vercel, tập trung vào caching và refetching tự động.
  - Ví dụ:
```javascript
import useSWR from 'swr';

const fetcher = (url) => fetch(url).then((res) => res.json());

function Component() {
  const { data, error } = useSWR('https://api.example.com/data', fetcher);

  if (error) return <div>Lỗi khi tải dữ liệu</div>;
  if (!data) return <div>Đang tải...</div>;

  return <div>{data.title}</div>;
}
```
- **React Query**:
  - Thư viện mạnh mẽ hơn, hỗ trợ các tính năng quản lý trạng thái dữ liệu phức tạp.
  - Ví dụ:
```javascript
import { useQuery } from '@tanstack/react-query';

function Component() {
  const { data, isLoading, error } = useQuery({
    queryKey: ['data'],
    queryFn: () => fetch('https://api.example.com/data').then((res) => res.json()),
  });

  if (isLoading) return <div>Đang tải...</div>;
  if (error) return <div>Lỗi: {error.message}</div>;

  return <div>{data.title}</div>;
}
```
- **Ưu điểm**:
  - Lý tưởng cho dữ liệu động, cụ thể theo người dùng (ví dụ: dữ liệu phiên người dùng).
  - Giảm tải máy chủ khi kết hợp với SSG hoặc SSR.
  - Hỗ trợ caching, refetching tự động, và xử lý lỗi.
- **Nhược điểm**:
  - Phụ thuộc vào JavaScript phía máy khách, có thể ảnh hưởng đến SEO nếu không kết hợp với pre-rendering.
  - Yêu cầu quản lý trạng thái phức tạp hơn so với SSR/SSG.
- **Trường hợp sử dụng**: Bảng điều khiển người dùng, nội dung thời gian thực, hoặc các thành phần cần cập nhật thường xuyên mà không yêu cầu SEO.