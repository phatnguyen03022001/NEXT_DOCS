# V. Styles & CSS trong Next.js

## 1. Hỗ trợ CSS Modules trong Next.js

- **Mục đích**: CSS Modules giúp viết CSS có phạm vi cục bộ (scoped locally), tránh xung đột kiểu dáng giữa các component.
- **Cách hoạt động**:
  - Các file CSS có đuôi `.module.css` (hoặc `.module.scss` cho SCSS) được Next.js xử lý tự động như CSS Modules.
  - Các class được tạo ra với tên duy nhất, đảm bảo không xung đột với các class khác.
- **Cách sử dụng**:
  - Tạo file, ví dụ: `styles/Home.module.css`.
  - Định nghĩa các class trong file CSS.
  - Nhập và sử dụng trong component.
- **Ví dụ**:
```css
/* styles/Home.module.css */
.container {
  padding: 20px;
  background-color: #f0f0f0;
}

.title {
  color: #333;
  font-size: 24px;
}
```
```javascript
// pages/index.js
import styles from '../styles/Home.module.css';

export default function Home() {
  return (
    <div className={styles.container}>
      <h1 className={styles.title}>Chào mừng đến với Next.js!</h1>
    </div>
  );
}
```
- **Ưu điểm**:
  - Phạm vi cục bộ, tránh xung đột tên class.
  - Dễ dàng tích hợp với JavaScript (có thể sử dụng biến động cho class).
  - Tự động tối ưu hóa và loại bỏ CSS không sử dụng trong production.
- **Nhược điểm**:
  - Cần đặt tên file với đuôi `.module.css` để kích hoạt.
  - Không phù hợp cho global styles (cần cấu hình riêng).
- **Trường hợp sử dụng**: Các component cần kiểu dáng riêng biệt, như UI module hóa.

## 2. Sử dụng Sass / SCSS

- **Mục đích**: Sử dụng Sass/SCSS để viết CSS với các tính năng nâng cao như biến, lồng (nesting), và mixin.
- **Cài đặt**:
  - Cài đặt package `sass`: `npm install sass`.
  - Next.js tích hợp sẵn hỗ trợ Sass, không cần cấu hình thêm.
- **Cách sử dụng**:
  - Tạo file `.scss` hoặc `.module.scss` (cho CSS Modules).
  - Nhập file SCSS vào component giống như CSS thông thường.
- **Ví dụ**:
```scss
/* styles/Home.module.scss */
$primary-color: #0070f3;

.container {
  padding: 20px;
  background-color: #f0f0f0;

  .title {
    color: $primary-color;
    font-size: 24px;
  }
}
```
```javascript
// pages/index.js
import styles from '../styles/Home.module.scss';

export default function Home() {
  return (
    <div className={styles.container}>
      <h1 className={styles.title}>Chào mừng đến với Next.js!</h1>
    </div>
  );
}
```
- **Ưu điểm**:
  - Hỗ trợ các tính năng mạnh mẽ như biến, lồng, và module hóa.
  - Tích hợp dễ dàng với CSS Modules.
  - Cộng đồng lớn, nhiều thư viện SCSS sẵn có.
- **Nhược điểm**:
  - Cần cài đặt thêm package `sass`.
  - Hiệu suất biên dịch chậm hơn CSS thông thường với dự án lớn.
- **Trường hợp sử dụng**: Các dự án cần cấu trúc CSS phức tạp, sử dụng biến và lồng để bảo trì dễ dàng.

## 3. Styled-components / Emotion / Tailwind CSS

### a. Styled-components
- **Mục đích**: Viết CSS-in-JS, cho phép định nghĩa kiểu dáng trong JavaScript, gắn trực tiếp vào component.
- **Cài đặt**:
  - Cài đặt: `npm install styled-components`.
  - Cần thêm plugin `babel-plugin-styled-components` để tối ưu hóa (tùy chọn).
- **Ví dụ**:
```javascript
// components/Button.js
import styled from 'styled-components';

const StyledButton = styled.button`
  background-color: #0070f3;
  color: white;
  padding: 10px 20px;
  border: none;
  border-radius: 5px;

  &:hover {
    background-color: #005bb5;
  }
`;

export default function Button() {
  return <StyledButton>Nhấn tôi</StyledButton>;
}
```
- **Ưu điểm**:
  - Kiểu dáng gắn liền với component, dễ quản lý.
  - Hỗ trợ props động để thay đổi kiểu dáng.
  - Tự động loại bỏ CSS không sử dụng.
- **Nhược điểm**:
  - Thêm phụ thuộc vào thư viện.
  - Hiệu suất có thể chậm hơn CSS thông thường với dự án lớn.
- **Trường hợp sử dụng**: Các ứng dụng cần kiểu dáng động, tích hợp chặt chẽ với logic JavaScript.

### b. Emotion
- **Mục đích**: Tương tự styled-components, cung cấp CSS-in-JS với hiệu suất tốt hơn và API linh hoạt.
- **Cài đặt**:
  - Cài đặt: `npm install @emotion/react @emotion/styled`.
- **Ví dụ**:
```javascript
// components/Button.js
import styled from '@emotion/styled';

const StyledButton = styled.button`
  background-color: #0070f3;
  color: white;
  padding: 10px 20px;
  border: none;
  border-radius: 5px;

  &:hover {
    background-color: #005bb5;
  }
`;

export default function Button() {
  return <StyledButton>Nhấn tôi</StyledButton>;
}
```
- **Ưu điểm**:
  - Hiệu suất tốt hơn styled-components.
  - Hỗ trợ cả styled API và css prop.
  - Nhẹ hơn, dễ tích hợp với Next.js.
- **Nhược điểm**:
  - Cộng đồng nhỏ hơn styled-components.
  - Cần làm quen với cú pháp css prop nếu sử dụng.
- **Trường hợp sử dụng**: Các dự án ưu tiên hiệu suất và cần CSS-in-JS linh hoạt.

### c. Tailwind CSS
- **Mục đích**: Sử dụng utility-first CSS framework để viết kiểu dáng trực tiếp trong HTML/JSX thông qua các class tiện ích.
- **Cài đặt**:
  - Cài đặt: `npm install -D tailwindcss postcss autoprefixer`.
  - Tạo file cấu hình: `npx tailwindcss init -p`.
  - Cấu hình `tailwind.config.js`:
```javascript
/** @type {import('tailwindcss').Config} */
module.exports = {
  content: [
    './pages/**/*.{js,ts,jsx,tsx}',
    './components/**/*.{js,ts,jsx,tsx}',
  ],
  theme: {
    extend: {},
  },
  plugins: [],
};
```
  - Thêm vào `styles/globals.css`:
```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```
- **Ví dụ**:
```javascript
// pages/index.js
export default function Home() {
  return (
    <div className="p-5 bg-gray-100">
      <h1 className="text-2xl text-blue-600">Chào mừng đến với Next.js!</h1>
      <button className="px-4 py-2 bg-blue-500 text-white rounded hover:bg-blue-700">
        Nhấn tôi
      </button>
    </div>
  );
}
```
- **Ưu điểm**:
  - Phát triển nhanh, không cần viết CSS riêng.
  - Tùy chỉnh dễ dàng thông qua file cấu hình.
  - Tích hợp tốt với Next.js và tối ưu hóa tree-shaking.
- **Nhược điểm**:
  - HTML/JSX có thể trở nên dài dòng với nhiều class.
  - Cần làm quen với hệ thống class của Tailwind.
- **Trường hợp sử dụng**: Các dự án cần phát triển nhanh, giao diện nhất quán, hoặc đội ngũ không muốn viết CSS phức tạp.

## 4. Cấu hình global styles

- **Mục đích**: Áp dụng kiểu dáng toàn cục cho toàn bộ ứng dụng, như font, màu sắc mặc định, hoặc reset CSS.
- **Cách thực hiện**:
  - Tạo file `styles/globals.css` (hoặc `.scss`).
  - Nhập file vào `_app.js` để áp dụng cho tất cả các trang.
- **Ví dụ**:
```css
/* styles/globals.css */
body {
  margin: 0;
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, sans-serif;
  background-color: #f9f9f9;
}

* {
  box-sizing: border-box;
}
```
```javascript
// pages/_app.js
import '../styles/globals.css';

export default function MyApp({ Component, pageProps }) {
  return <Component {...pageProps} />;
}
```
- **Sử dụng với Tailwind CSS**:
  - Trong `globals.css`, thêm các directive của Tailwind và tùy chỉnh thêm nếu cần:
```css
/* styles/globals.css */
@tailwind base;
@tailwind components;
@tailwind utilities;

body {
  background-color: #f9f9f9;
}
```
- **Sử dụng với CSS-in-JS (styled-components/Emotion)**:
  - Sử dụng `Global` component hoặc `createGlobalStyle` để định nghĩa global styles.
- **Ví dụ với styled-components**:
```javascript
// styles/GlobalStyles.js
import { createGlobalStyle } from 'styled-components';

const GlobalStyles = createGlobalStyle`
  body {
    margin: 0;
    font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, sans-serif;
    background-color: #f9f9f9;
  }

  * {
    box-sizing: border-box;
  }
`;

export default GlobalStyles;
```
```javascript
// pages/_app.js
import GlobalStyles from '../styles/GlobalStyles';

export default function MyApp({ Component, pageProps }) {
  return (
    <>
      <GlobalStyles />
      <Component {...pageProps} />
    </>
  );
}
```
- **Lưu ý**:
  - Global styles nên được giữ tối thiểu để tránh xung đột với CSS Modules hoặc CSS-in-JS.
  - Sử dụng `normalize.css` hoặc reset CSS nếu cần chuẩn hóa kiểu dáng trên các trình duyệt.
  - Với Tailwind, tận dụng `@layer` để quản lý base styles hoặc components.
- **Trường hợp sử dụng**: Thiết lập kiểu dáng mặc định cho toàn ứng dụng, như font, margin, hoặc màu nền.