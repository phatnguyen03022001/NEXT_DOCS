# XII. Dự án thực tế trong Next.js

## 1. Tạo blog cá nhân với Markdown và SSG

- **Mục đích**: Xây dựng một blog cá nhân sử dụng Static Site Generation (SSG) và các file Markdown để quản lý nội dung bài viết.
- **Cách thực hiện**:
  1. Sử dụng thư viện như `gray-matter` để đọc file Markdown và `marked` để chuyển đổi Markdown sang HTML.
  2. Tạo các trang tĩnh với `getStaticProps` và `getStaticPaths`.
  3. Tạo giao diện đơn giản cho danh sách bài viết và chi tiết bài viết.
- **Cài đặt**:
  - Cài đặt các thư viện cần thiết: `npm install gray-matter marked`
- **Cấu trúc thư mục**:
```
project/
  pages/
    posts/
      [slug].js
    index.js
  posts/
    first-post.md
    second-post.md
  styles/
    Blog.module.css
```
- **Ví dụ code**:
  - File Markdown (`posts/first-post.md`):
```markdown
---
title: Bài viết đầu tiên
date: 2023-10-01
---
# Chào mừng đến với bài viết đầu tiên

Đây là nội dung bài viết được viết bằng Markdown.
```
  - Trang danh sách bài viết (`pages/index.js`):
```javascript
import fs from 'fs';
import path from 'path';
import matter from 'gray-matter';
import Link from 'next/link';

export default function Home({ posts }) {
  return (
    <div>
      <h1>Blog cá nhân</h1>
      <ul>
        {posts.map((post) => (
          <li key={post.slug}>
            <Link href={`/posts/${post.slug}`}>
              {post.frontmatter.title}
            </Link>
          </li>
        ))}
      </ul>
    </div>
  );
}

export async function getStaticProps() {
  const postsDirectory = path.join(process.cwd(), 'posts');
  const filenames = fs.readdirSync(postsDirectory);

  const posts = filenames.map((filename) => {
    const filePath = path.join(postsDirectory, filename);
    const fileContents = fs.readFileSync(filePath, 'utf8');
    const { data: frontmatter } = matter(fileContents);
    return {
      slug: filename.replace('.md', ''),
      frontmatter,
    };
  });

  return {
    props: {
      posts,
    },
  };
}
```
  - Trang chi tiết bài viết (`pages/posts/[slug].js`):
```javascript
import fs from 'fs';
import path from 'path';
import matter from 'gray-matter';
import { marked } from 'marked';

export default function Post({ frontmatter, content }) {
  return (
    <div>
      <h1>{frontmatter.title}</h1>
      <p>{frontmatter.date}</p>
      <div dangerouslySetInnerHTML={{ __html: marked(content) }} />
    </div>
  );
}

export async function getStaticProps({ params }) {
  const filePath = path.join(process.cwd(), 'posts', `${params.slug}.md`);
  const fileContents = fs.readFileSync(filePath, 'utf8');
  const { data: frontmatter, content } = matter(fileContents);

  return {
    props: {
      frontmatter,
      content,
    },
  };
}

export async function getStaticPaths() {
  const postsDirectory = path.join(process.cwd(), 'posts');
  const filenames = fs.readdirSync(postsDirectory);
  const paths = filenames.map((filename) => ({
    params: { slug: filename.replace('.md', '') },
  }));

  return {
    paths,
    fallback: false,
  };
}
```
- **Ưu điểm**:
  - Tối ưu SEO với SSG.
  - Quản lý nội dung đơn giản với Markdown.
  - Hiệu suất cao nhờ trang tĩnh.
- **Nhược điểm**:
  - Không phù hợp cho nội dung động hoặc cần cập nhật thường xuyên.
  - Cần thêm thư viện để xử lý Markdown.
- **Trường hợp sử dụng**: Blog cá nhân, trang tài liệu, hoặc portfolio tĩnh.

## 2. Xây dựng trang thương mại điện tử đơn giản

- **Mục đích**: Xây dựng một trang thương mại điện tử cơ bản với danh sách sản phẩm, chi tiết sản phẩm, và giỏ hàng sử dụng SSG hoặc SSR.
- **Cách thực hiện**:
  1. Sử dụng `getStaticProps` để lấy danh sách sản phẩm từ API hoặc cơ sở dữ liệu.
  2. Tạo trang chi tiết sản phẩm với route động (`[id].js`).
  3. Quản lý giỏ hàng bằng trạng thái cục bộ hoặc Context API.
- **Ví dụ code**:
  - Trang danh sách sản phẩm (`pages/index.js`):
```javascript
import Link from 'next/link';

export default function Home({ products }) {
  return (
    <div>
      <h1>Cửa hàng</h1>
      <ul>
        {products.map((product) => (
          <li key={product.id}>
            <Link href={`/products/${product.id}`}>
              {product.name} - {product.price} VNĐ
            </Link>
          </li>
        ))}
      </ul>
    </div>
  );
}

export async function getStaticProps() {
  const res = await fetch('https://api.example.com/products');
  const products = await res.json();

  return {
    props: {
      products,
    },
    revalidate: 60, // ISR: Cập nhật sau 60 giây
  };
}
```
  - Trang chi tiết sản phẩm (`pages/products/[id].js`):
```javascript
export default function Product({ product }) {
  return (
    <div>
      <h1>{product.name}</h1>
      <p>Giá: {product.price} VNĐ</p>
      <p>{product.description}</p>
      <button>Thêm vào giỏ hàng</button>
    </div>
  );
}

export async function getStaticProps({ params }) {
  const res = await fetch(`https://api.example.com/products/${params.id}`);
  const product = await res.json();

  return {
    props: {
      product,
    },
    revalidate: 60,
  };
}

export async function getStaticPaths() {
  const res = await fetch('https://api.example.com/products');
  const products = await res.json();
  const paths = products.map((product) => ({
    params: { id: product.id.toString() },
  }));

  return {
    paths,
    fallback: 'blocking',
  };
}
```
  - Giỏ hàng với Context API (`context/CartContext.js`):
```javascript
import { createContext, useContext, useState } from 'react';

const CartContext = createContext();

export function CartProvider({ children }) {
  const [cart, setCart] = useState([]);

  const addToCart = (product) => {
    setCart([...cart, product]);
  };

  return (
    <CartContext.Provider value={{ cart, addToCart }}>
      {children}
    </CartContext.Provider>
  );
}

export function useCart() {
  return useContext(CartContext);
}
```
```javascript
// pages/_app.js
import { CartProvider } from '../context/CartContext';

export default function MyApp({ Component, pageProps }) {
  return (
    <CartProvider>
      <Component {...pageProps} />
    </CartProvider>
  );
}
```
```javascript
// components/Cart.js
import { useCart } from '../context/CartContext';

export default function Cart() {
  const { cart } = useCart();

  return (
    <div>
      <h1>Giỏ hàng</h1>
      <ul>
        {cart.map((item, index) => (
          <li key={index}>{item.name} - {item.price} VNĐ</li>
        ))}
      </ul>
    </div>
  );
}
```
- **Ưu điểm**:
  - Tối ưu SEO và hiệu suất với SSG/ISR.
  - Dễ mở rộng với các tính năng như thanh toán hoặc xác thực.
- **Nhược điểm**:
  - Cần tích hợp thêm API thanh toán cho chức năng đầy đủ.
  - Quản lý giỏ hàng phức tạp hơn với ứng dụng lớn.
- **Trường hợp sử dụng**: Cửa hàng trực tuyến nhỏ, trang bán hàng sản phẩm đơn giản.

## 3. Dashboard quản trị với xác thực

- **Mục đích**: Xây dựng dashboard quản trị với xác thực người dùng, sử dụng NextAuth.js và bảo vệ route.
- **Cách thực hiện**:
  1. Sử dụng NextAuth.js để xác thực.
  2. Bảo vệ các route dashboard bằng middleware hoặc `getServerSideProps`.
  3. Hiển thị dữ liệu quản trị (ví dụ: danh sách người dùng).
- **Ví dụ code**:
  - Cấu hình NextAuth.js (`pages/api/auth/[...nextauth].js`):
```javascript
import NextAuth from 'next-auth';
import CredentialsProvider from 'next-auth/providers/credentials';

export default NextAuth({
  providers: [
    CredentialsProvider({
      name: 'Credentials',
      credentials: {
        username: { label: 'Tên người dùng', type: 'text' },
        password: { label: 'Mật khẩu', type: 'password' },
      },
      async authorize(credentials) {
        if (credentials.username === 'admin' && credentials.password === 'password') {
          return { id: 1, name: 'Admin', email: 'admin@example.com', role: 'admin' };
        }
        return null;
      },
    }),
  ],
  callbacks: {
    async session({ session, token }) {
      session.user.role = token.role || 'user';
      return session;
    },
  },
});
```
  - Bảo vệ route dashboard (`middleware.js`):
```javascript
import { getServerSession } from 'next-auth';
import { NextResponse } from 'next/server';
import { authOptions } from './pages/api/auth/[...nextauth]';

export async function middleware(req) {
  const session = await getServerSession(req, NextResponse.next(), authOptions);

  if (!session || session.user.role !== 'admin') {
    return NextResponse.redirect(new URL('/login', req.url));
  }

  return NextResponse.next();
}

export const config = {
  matcher: ['/admin/:path*'],
};
```
  - Trang dashboard (`pages/admin/index.js`):
```javascript
import { getServerSession } from 'next-auth';
import { authOptions } from '../api/auth/[...nextauth]';

export default function AdminDashboard({ users }) {
  return (
    <div>
      <h1>Dashboard Quản trị</h1>
      <ul>
        {users.map((user) => (
          <li key={user.id}>{user.name} - {user.email}</li>
        ))}
      </ul>
    </div>
  );
}

export async function getServerSideProps(context) {
  const session = await getServerSession(context.req, context.res, authOptions);

  if (!session || session.user.role !== 'admin') {
    return {
      redirect: {
        destination: '/login',
        permanent: false,
      },
    };
  }

  const users = await fetch('https://api.example.com/users').then((res) => res.json());

  return {
    props: {
      users,
    },
  };
}
```
- **Ưu điểm**:
  - Bảo mật cao với xác thực và phân quyền.
  - Dễ tích hợp với các API để quản lý dữ liệu.
- **Nhược điểm**:
  - Cần cấu hình xác thực và bảo vệ route cẩn thận.
  - Có thể phức tạp khi tích hợp với nhiều nguồn dữ liệu.
- **Trường hợp sử dụng**: Hệ thống quản trị người dùng, sản phẩm, hoặc nội dung.

## 4. Kết nối với CMS (Sanity, Strapi, Contentful...)

- **Mục đích**: Kết nối Next.js với CMS để quản lý nội dung động, như bài viết, sản phẩm, hoặc trang tĩnh.
- **Cách thực hiện**:
  1. Cài đặt SDK của CMS (ví dụ: `@sanity/client`, `contentful`, hoặc sử dụng fetch cho Strapi).
  2. Sử dụng `getStaticProps` hoặc `getServerSideProps` để lấy dữ liệu từ CMS.
- **Ví dụ với Sanity**:
  - Cài đặt: `npm install @sanity/client`
  - Cấu hình client:
```javascript
// lib/sanity.js
import sanityClient from '@sanity/client';

export const client = sanityClient({
  projectId: 'your-project-id',
  dataset: 'production',
  useCdn: true,
});
```
  - Lấy dữ liệu trong `getStaticProps`:
```javascript
// pages/index.js
import { client } from '../lib/sanity';

export default function Home({ posts }) {
  return (
    <div>
      <h1>Blog từ Sanity</h1>
      <ul>
        {posts.map((post) => (
          <li key={post._id}>{post.title}</li>
        ))}
      </ul>
    </div>
  );
}

export async function getStaticProps() {
  const posts = await client.fetch('*[_type == "post"]{ _id, title }');

  return {
    props: {
      posts,
    },
    revalidate: 60,
  };
}
```
- **Ví dụ với Strapi**:
  - Lấy dữ liệu bằng fetch:
```javascript
// pages/index.js
export default function Home({ posts }) {
  return (
    <div>
      <h1>Blog từ Strapi</h1>
      <ul>
        {posts.map((post) => (
          <li key={post.id}>{post.attributes.title}</li>
        ))}
      </ul>
    </div>
  );
}

export async function getStaticProps() {
  const res = await fetch('http://localhost:1337/api/posts');
  const { data: posts } = await res.json();

  return {
    props: {
      posts,
    },
    revalidate: 60,
  };
}
```
- **Ưu điểm**:
  - Dễ dàng quản lý nội dung qua giao diện CMS.
  - Tích hợp tốt với SSG/ISR để tối ưu hiệu suất.
- **Nhược điểm**:
  - Cần cấu hình CMS và API key.
  - Có thể tăng chi phí nếu sử dụng CMS trả phí như Contentful.
- **Trường hợp sử dụng**: Blog, trang thương mại điện tử, hoặc trang web cần quản lý nội dung động.

## 5. Kết nối với GraphQL API (Apollo Client)

- **Mục đích**: Kết nối Next.js với API GraphQL để lấy dữ liệu động, sử dụng Apollo Client.
- **Cài đặt**:
  - Cài đặt: `npm install @apollo/client graphql`
- **Cấu hình Apollo Client**:
```javascript
// lib/apollo.js
import { ApolloClient, InMemoryCache, HttpLink } from '@apollo/client';

export const client = new ApolloClient({
  link: new HttpLink({
    uri: 'https://api.example.com/graphql',
  }),
  cache: new InMemoryCache(),
});
```
- **Ví dụ sử dụng trong SSG**:
```javascript
// pages/index.js
import { gql } from '@apollo/client';
import { client } from '../lib/apollo';

export default function Home({ posts }) {
  return (
    <div>
      <h1>Blog từ GraphQL</h1>
      <ul>
        {posts.map((post) => (
          <li key={post.id}>{post.title}</li>
        ))}
      </ul>
    </div>
  );
}

export async function getStaticProps() {
  const { data } = await client.query({
    query: gql`
      query GetPosts {
        posts {
          id
          title
        }
      }
    `,
  });

  return {
    props: {
      posts: data.posts,
    },
    revalidate: 60,
  };
}
```
- **Ví dụ sử dụng trên client-side**:
```javascript
// components/PostList.js
import { useQuery, gql } from '@apollo/client';

const GET_POSTS = gql`
  query GetPosts {
    posts {
      id
      title
    }
  }
`;

export default function PostList() {
  const { loading, error, data } = useQuery(GET_POSTS);

  if (loading) return <p>Đang tải...</p>;
  if (error) return <p>Lỗi: {error.message}</p>;

  return (
    <ul>
      {data.posts.map((post) => (
        <li key={post.id}>{post.title}</li>
      ))}
    </ul>
  );
}
```
- **Ưu điểm**:
  - Linh hoạt trong việc lấy dữ liệu với GraphQL.
  - Hỗ trợ cả client-side và server-side fetching.
  - Tích hợp tốt với các tính năng Next.js như SSG/ISR.
- **Nhược điểm**:
  - Cần cấu hình Apollo Client và hiểu cú pháp GraphQL.
  - Thêm phụ thuộc và có thể làm tăng kích thước bundle.
- **Trường hợp sử dụng**: Ứng dụng cần lấy dữ liệu từ API GraphQL, như CMS hoặc hệ thống phức tạp.