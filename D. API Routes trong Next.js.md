# IV. API Routes trong Next.js

## 1. Tạo API đơn giản với /pages/api

- **Mục đích**: Next.js cho phép tạo các API endpoints dễ dàng bằng cách sử dụng thư mục `/pages/api`. Mỗi file trong thư mục này sẽ tự động trở thành một API route.
- **Cách hoạt động**: 
  - Các file trong `/pages/api` được ánh xạ thành các endpoint `/api/*`.
  - Mỗi file xuất một hàm xử lý (handler) mặc định nhận các tham số `req` (request) và `res` (response).
- **Ví dụ**:
```javascript
// pages/api/hello.js
export default function handler(req, res) {
  res.status(200).json({ message: 'Xin chào từ API!' });
}
```
- **Kết quả**: Truy cập `/api/hello` sẽ trả về JSON: `{ "message": "Xin chào từ API!" }`.
- **Lưu ý**:
  - File phải xuất một hàm mặc định.
  - Không cần cấu hình server riêng; Next.js tự xử lý routing.
- **Trường hợp sử dụng**: Xử lý các yêu cầu API đơn giản, như lấy dữ liệu, xác thực người dùng, hoặc tích hợp với cơ sở dữ liệu.

## 2. Xử lý request và response (GET, POST, PUT, DELETE)

- **Mục đích**: Xử lý các phương thức HTTP khác nhau (GET, POST, PUT, DELETE) trong API route để thực hiện các thao tác CRUD (Create, Read, Update, Delete).
- **Cách hoạt động**:
  - Sử dụng thuộc tính `req.method` để kiểm tra loại yêu cầu HTTP.
  - Sử dụng `res` để trả về trạng thái HTTP và dữ liệu (thường là JSON).
- **Ví dụ**:
```javascript
// pages/api/users/[id].js
export default function handler(req, res) {
  const { method } = req;
  const { id } = req.query; // Lấy tham số động từ URL

  switch (method) {
    case 'GET':
      res.status(200).json({ id, name: `Người dùng ${id}` });
      break;
    case 'POST':
      const { name } = req.body; // Lấy dữ liệu từ body
      res.status(201).json({ id, name });
      break;
    case 'PUT':
      const { updatedName } = req.body;
      res.status(200).json({ id, name: updatedName });
      break;
    case 'DELETE':
      res.status(200).json({ message: `Xóa người dùng ${id}` });
      break;
    default:
      res.setHeader('Allow', ['GET', 'POST', 'PUT', 'DELETE']);
      res.status(405).end(`Phương thức ${method} không được hỗ trợ`);
  }
}
```
- **Giải thích**:
  - **GET**: Lấy thông tin người dùng dựa trên `id`.
  - **POST**: Tạo mới người dùng với dữ liệu từ `req.body`.
  - **PUT**: Cập nhật thông tin người dùng.
  - **DELETE**: Xóa người dùng.
  - **Mã lỗi 405**: Trả về nếu phương thức HTTP không được hỗ trợ.
- **Lưu ý**:
  - Sử dụng `req.query` để lấy tham số từ URL (ví dụ: `/api/users/1` → `id = 1`).
  - Sử dụng `req.body` để lấy dữ liệu từ body của yêu cầu (cần parse JSON nếu gửi dạng JSON).
- **Trường hợp sử dụng**: Xây dựng các API CRUD cho ứng dụng, như quản lý danh sách sản phẩm hoặc người dùng.

## 3. Cấu trúc RESTful API cơ bản

- **Định nghĩa**: RESTful API là cách thiết kế API dựa trên các nguyên tắc REST (Representational State Transfer), sử dụng các phương thức HTTP chuẩn và cấu trúc URL rõ ràng.
- **Nguyên tắc cơ bản**:
  - **Tài nguyên (Resources)**: Mỗi endpoint đại diện cho một tài nguyên (ví dụ: `/api/users` cho người dùng).
  - **Phương thức HTTP**:
    - `GET`: Lấy dữ liệu (read).
    - `POST`: Tạo mới (create).
    - `PUT`/`PATCH`: Cập nhật (update).
    - `DELETE`: Xóa (delete).
  - **URL có ý nghĩa**: Sử dụng danh từ (noun) cho tài nguyên, ví dụ: `/api/users` thay vì `/api/getUsers`.
  - **Trạng thái HTTP**:
    - `200 OK`: Yêu cầu thành công.
    - `201 Created`: Tạo tài nguyên thành công.
    - `400 Bad Request`: Yêu cầu không hợp lệ.
    - `404 Not Found`: Tài nguyên không tồn tại.
    - `500 Internal Server Error`: Lỗi máy chủ.
- **Ví dụ cấu trúc RESTful**:
  - `GET /api/users`: Lấy danh sách người dùng.
  - `GET /api/users/1`: Lấy thông tin người dùng với ID = 1.
  - `POST /api/users`: Tạo người dùng mới.
  - `PUT /api/users/1`: Cập nhật người dùng với ID = 1.
  - `DELETE /api/users/1`: Xóa người dùng với ID = 1.
- **Lưu ý**:
  - Giữ URL ngắn gọn, rõ ràng và dễ hiểu.
  - Trả về dữ liệu JSON để đảm bảo tính tương thích.
  - Xử lý lỗi một cách rõ ràng với mã trạng thái phù hợp.
- **Trường hợp sử dụng**: Xây dựng API có cấu trúc dễ bảo trì, tích hợp với frontend hoặc ứng dụng bên thứ ba.

## 4. Gọi API từ frontend (Axios, Fetch, SWR)

- **Mục đích**: Gọi API từ phía máy khách (frontend) để lấy hoặc gửi dữ liệu, sử dụng các công cụ như `fetch`, `Axios`, hoặc `SWR`.
- **Các phương pháp**:

### a. Sử dụng Fetch (API tích hợp sẵn)
- **Cách hoạt động**: Sử dụng API `fetch` của trình duyệt để gửi yêu cầu HTTP.
- **Ví dụ**:
```javascript
async function fetchData() {
  try {
    const response = await fetch('/api/users/1');
    const data = await response.json();
    console.log(data);
  } catch (error) {
    console.error('Lỗi:', error);
  }
}
```
- **Ưu điểm**: Tích hợp sẵn, không cần cài đặt thêm thư viện.
- **Nhược điểm**: Cần xử lý lỗi và cấu hình thủ công (ví dụ: headers, JSON parsing).

### b. Sử dụng Axios
- **Cách hoạt động**: Thư viện HTTP client phổ biến, hỗ trợ cấu hình dễ dàng và xử lý lỗi tốt hơn.
- **Cài đặt**: `npm install axios`
- **Ví dụ**:
```javascript
import axios from 'axios';

async function fetchData() {
  try {
    const response = await axios.get('/api/users/1');
    console.log(response.data);
  } catch (error) {
    console.error('Lỗi:', error.response?.data || error.message);
  }
}

// Gửi POST request
async function createUser() {
  try {
    const response = await axios.post('/api/users', { name: 'Người dùng mới' });
    console.log(response.data);
  } catch (error) {
    console.error('Lỗi:', error.response?.data || error.message);
  }
}
```
- **Ưu điểm**:
  - Xử lý JSON tự động.
  - Hỗ trợ cấu hình headers, timeout, và xử lý lỗi dễ dàng.
  - Hỗ trợ các phương thức HTTP đầy đủ.
- **Nhược điểm**: Cần cài đặt thư viện bổ sung.

### c. Sử dụng SWR
- **Cách hoạt động**: Thư viện từ Vercel, tối ưu cho việc lấy dữ liệu phía máy khách với caching và refetching tự động.
- **Cài đặt**: `npm install swr`
- **Ví dụ**:
```javascript
import useSWR from 'swr';

const fetcher = (url) => fetch(url).then((res) => res.json());

function UserComponent() {
  const { data, error } = useSWR('/api/users/1', fetcher);

  if (error) return <div>Lỗi khi tải dữ liệu</div>;
  if (!data) return <div>Đang tải...</div>;

  return <div>Tên: {data.name}</div>;
}
```
- **Ưu điểm**:
  - Tự động caching và refetching (ví dụ: khi component được focus lại).
  - Xử lý trạng thái tải và lỗi dễ dàng.
  - Phù hợp với các ứng dụng cần dữ liệu động.
- **Nhược điểm**: Thêm phụ thuộc vào thư viện và cần hiểu cách hoạt động của caching.

- **Lưu ý khi gọi API**:
  - **Base URL**: Khi gọi API trong cùng dự án Next.js, sử dụng đường dẫn tương đối (ví dụ: `/api/users`) thay vì URL đầy đủ.
  - **Xử lý lỗi**: Luôn kiểm tra và xử lý lỗi để cải thiện trải nghiệm người dùng.
  - **Tối ưu hiệu suất**: Kết hợp với SSG/SSR để giảm số lượng yêu cầu API từ máy khách.
- **Trường hợp sử dụng**:
  - `Fetch`: Các dự án nhỏ, không muốn thêm phụ thuộc.
  - `Axios`: Các ứng dụng cần gọi API phức tạp hoặc tích hợp với nhiều API bên ngoài.
  - `SWR`: Các ứng dụng cần dữ liệu động với hiệu suất cao và trải nghiệm người dùng mượt mà.