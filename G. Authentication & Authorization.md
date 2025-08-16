# VII. Authentication & Authorization trong Next.js

## 1. Cơ bản về xác thực trong Next.js

- **Mục đích**: Xác thực (authentication) xác minh danh tính người dùng, còn phân quyền (authorization) kiểm soát quyền truy cập vào tài nguyên hoặc chức năng.
- **Cách hoạt động**:
  - **Authentication**: Xác định "người dùng là ai" thông qua đăng nhập (email/mật khẩu, OAuth, token, v.v.).
  - **Authorization**: Quyết định "người dùng được làm gì" dựa trên vai trò hoặc quyền.
- **Các phương pháp xác thực phổ biến**:
  - **Email/Mật khẩu**: Sử dụng cơ sở dữ liệu để lưu trữ và kiểm tra thông tin đăng nhập.
  - **OAuth**: Sử dụng nhà cung cấp bên thứ ba (Google, GitHub, Facebook, v.v.).
  - **JWT (JSON Web Token)**: Tạo token để xác thực người dùng trên client và server.
- **Tích hợp trong Next.js**:
  - Sử dụng API Routes để xử lý đăng nhập/đăng xuất.
  - Kết hợp với thư viện như NextAuth.js để đơn giản hóa quá trình xác thực.
  - Lưu trữ trạng thái xác thực (session) bằng cookie, localStorage, hoặc server-side session.
- **Trường hợp sử dụng**: Đăng nhập người dùng, bảo vệ các trang nhạy cảm, hoặc tích hợp với dịch vụ bên thứ ba.

## 2. Sử dụng NextAuth.js để auth (Google, GitHub, Email, v.v.)

- **Mục đích**: NextAuth.js là thư viện xác thực mạnh mẽ, dễ sử dụng cho Next.js, hỗ trợ nhiều nhà cung cấp (Google, GitHub, Email, v.v.) và quản lý session.
- **Cài đặt**:
  - Cài đặt: `npm install next-auth`
  - Tạo file cấu hình tại `pages/api/auth/[...nextauth].js`.
- **Cấu hình NextAuth.js**:
```javascript
// pages/api/auth/[...nextauth].js
import NextAuth from 'next-auth';
import GoogleProvider from 'next-auth/providers/google';
import GitHubProvider from 'next-auth/providers/github';
import CredentialsProvider from 'next-auth/providers/credentials';

export default NextAuth({
  providers: [
    GoogleProvider({
      clientId: process.env.GOOGLE_CLIENT_ID,
      clientSecret: process.env.GOOGLE_CLIENT_SECRET,
    }),
    GitHubProvider({
      clientId: process.env.GITHUB_CLIENT_ID,
      clientSecret: process.env.GITHUB_CLIENT_SECRET,
    }),
    CredentialsProvider({
      name: 'Credentials',
      credentials: {
        username: { label: 'Tên người dùng', type: 'text' },
        password: { label: 'Mật khẩu', type: 'password' },
      },
      async authorize(credentials) {
        // Logic kiểm tra thông tin đăng nhập
        if (credentials.username === 'user' && credentials.password === 'password') {
          return { id: 1, name: 'Người dùng', email: 'user@example.com' };
        }
        return null;
      },
    }),
  ],
  pages: {
    signIn: '/auth/signin', // Trang đăng nhập tùy chỉnh
  },
  session: {
    strategy: 'jwt', // Sử dụng JWT để quản lý session
  },
  callbacks: {
    async session({ session, token }) {
      session.user.id = token.sub; // Thêm ID người dùng vào session
      return session;
    },
  },
});
```
- **Sử dụng trong ứng dụng**:
```javascript
// pages/auth/signin.js
import { signIn } from 'next-auth/react';

export default function SignIn() {
  return (
    <div>
      <h1>Đăng nhập</h1>
      <button onClick={() => signIn('google')}>Đăng nhập bằng Google</button>
      <button onClick={() => signIn('github')}>Đăng nhập bằng GitHub</button>
      <form
        onSubmit={(e) => {
          e.preventDefault();
          signIn('credentials', {
            username: e.target.username.value,
            password: e.target.password.value,
            callbackUrl: '/',
          });
        }}
      >
        <input name="username" type="text" placeholder="Tên người dùng" />
        <input name="password" type="password" placeholder="Mật khẩu" />
        <button type="submit">Đăng nhập</button>
      </form>
    </div>
  );
}
```
- **Lấy thông tin session**:
```javascript
// components/Profile.js
import { useSession, signOut } from 'next-auth/react';

export default function Profile() {
  const { data: session, status } = useSession();

  if (status === 'loading') return <div>Đang tải...</div>;
  if (!session) return <div>Vui lòng đăng nhập</div>;

  return (
    <div>
      <p>Xin chào, {session.user.name}</p>
      <button onClick={() => signOut()}>Đăng xuất</button>
    </div>
  );
}
```
- **Ưu điểm**:
  - Hỗ trợ nhiều nhà cung cấp xác thực (OAuth, email, credentials).
  - Tích hợp dễ dàng với Next.js (API Routes, SSR, SSG).
  - Quản lý session tự động (JWT hoặc database).
- **Nhược điểm**:
  - Cần cấu hình thêm cho các nhà cung cấp (client ID, secret).
  - Có thể phức tạp khi tùy chỉnh logic xác thực nâng cao.
- **Trường hợp sử dụng**: Ứng dụng cần đăng nhập nhanh qua Google, GitHub, hoặc email/mật khẩu.

## 3. Bảo vệ route (Protected Routes)

- **Mục đích**: Hạn chế truy cập vào các trang hoặc API chỉ dành cho người dùng đã xác thực.
- **Cách thực hiện**:
  - **Client-side**: Sử dụng `useSession` từ NextAuth.js để kiểm tra trạng thái đăng nhập.
  - **Server-side**: Sử dụng `getServerSession` trong `getServerSideProps` để kiểm tra session.
- **Ví dụ Client-side**:
```javascript
// pages/protected.js
import { useSession, signIn } from 'next-auth/react';

export default function ProtectedPage() {
  const { data: session, status } = useSession();

  if (status === 'loading') return <div>Đang tải...</div>;
  if (!session) {
    signIn(); // Chuyển hướng đến trang đăng nhập
    return null;
  }

  return <div>Đây là trang được bảo vệ. Chào, {session.user.name}!</div>;
}
```
- **Ví dụ Server-side**:
```javascript
// pages/protected.js
import { getServerSession } from 'next-auth';
import { authOptions } from './api/auth/[...nextauth]';

export default function ProtectedPage({ user }) {
  return <div>Đây là trang được bảo vệ. Chào, {user.name}!</div>;
}

export async function getServerSideProps(context) {
  const session = await getServerSession(context.req, context.res, authOptions);

  if (!session) {
    return {
      redirect: {
        destination: '/auth/signin',
        permanent: false,
      },
    };
  }

  return {
    props: {
      user: session.user,
    },
  };
}
```
- **Ưu điểm**:
  - Bảo vệ route cả ở client và server.
  - Tích hợp mượt mà với NextAuth.js.
  - Ngăn chặn hiển thị nội dung nhạy cảm trước khi kiểm tra xác thực.
- **Nhược điểm**:
  - Cần kiểm tra session ở cả client và server để đảm bảo bảo mật.
  - Có thể làm tăng độ phức tạp của code.
- **Trường hợp sử dụng**: Các trang nhạy cảm như hồ sơ người dùng, bảng điều khiển quản trị, hoặc nội dung trả phí.

## 4. Role-based Access Control (RBAC)

- **Mục đích**: Phân quyền truy cập dựa trên vai trò (role) của người dùng, như admin, user, hoặc guest.
- **Cách thực hiện**:
  - Lưu trữ vai trò trong session hoặc cơ sở dữ liệu.
  - Kiểm tra vai trò trong component hoặc `getServerSideProps` để giới hạn truy cập.
  - Sử dụng middleware hoặc logic tùy chỉnh để kiểm soát quyền.
- **Ví dụ với NextAuth.js**:
  - Thêm vai trò vào session trong callback:
```javascript
// pages/api/auth/[...nextauth].js
import NextAuth from 'next-auth';
import GoogleProvider from 'next-auth/providers/google';

export default NextAuth({
  providers: [
    GoogleProvider({
      clientId: process.env.GOOGLE_CLIENT_ID,
      clientSecret: process.env.GOOGLE_CLIENT_SECRET,
    }),
  ],
  callbacks: {
    async session({ session, token }) {
      session.user.id = token.sub;
      session.user.role = 'admin'; // Giả sử lấy từ DB hoặc logic khác
      return session;
    },
  },
});
```
  - Bảo vệ route dựa trên vai trò:
```javascript
// pages/admin.js
import { getServerSession } from 'next-auth';
import { authOptions } from './api/auth/[...nextauth]';

export default function AdminPage({ user }) {
  return <div>Chào mừng đến với trang quản trị, {user.name}!</div>;
}

export async function getServerSideProps(context) {
  const session = await getServerSession(context.req, context.res, authOptions);

  if (!session) {
    return {
      redirect: {
        destination: '/auth/signin',
        permanent: false,
      },
    };
  }

  if (session.user.role !== 'admin') {
    return {
      redirect: {
        destination: '/403', // Trang lỗi không có quyền
        permanent: false,
      },
    };
  }

  return {
    props: {
      user: session.user,
    },
  };
}
```
- **Sử dụng Middleware (Next.js Middleware)**:
  - Tạo file `middleware.js` để kiểm tra quyền truy cập:
```javascript
// middleware.js
import { getServerSession } from 'next-auth';
import { authOptions } from './pages/api/auth/[...nextauth]';
import { NextResponse } from 'next/server';

export async function middleware(req) {
  const session = await getServerSession(req, NextResponse.next(), authOptions);

  if (!session) {
    return NextResponse.redirect(new URL('/auth/signin', req.url));
  }

  if (req.nextUrl.pathname.startsWith('/admin') && session.user.role !== 'admin') {
    return NextResponse.redirect(new URL('/403', req.url));
  }

  return NextResponse.next();
}

export const config = {
  matcher: ['/protected/:path*', '/admin/:path*'],
};
```
- **Ưu điểm**:
  - Kiểm soát quyền truy cập chi tiết dựa trên vai trò.
  - Tích hợp dễ dàng với NextAuth.js và middleware.
  - Phù hợp cho ứng dụng có nhiều loại người dùng (admin, editor, user).
- **Nhược điểm**:
  - Cần quản lý vai trò trong cơ sở dữ liệu hoặc session.
  - Tăng độ phức tạp khi có nhiều vai trò hoặc quy tắc phân quyền.
- **Trường hợp sử dụng**: Ứng dụng với nhiều cấp độ truy cập, như trang quản trị, nội dung trả phí, hoặc hệ thống quản lý người dùng.