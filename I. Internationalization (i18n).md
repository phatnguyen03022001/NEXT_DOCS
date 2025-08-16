# IX. Internationalization (i18n) trong Next.js

## 1. Cấu hình đa ngôn ngữ trong Next.js

- **Mục đích**: Hỗ trợ đa ngôn ngữ (i18n) để cung cấp nội dung cho người dùng ở các ngôn ngữ và khu vực khác nhau.
- **Cách hoạt động**:
  - Next.js tích hợp sẵn hỗ trợ i18n thông qua cấu hình trong `next.config.js`.
  - Cho phép định nghĩa các ngôn ngữ (locales), ngôn ngữ mặc định, và cách xử lý routing.
- **Cấu hình**:
  - Thêm cấu hình i18n vào `next.config.js`:
```javascript
/** @type {import('next').NextConfig} */
module.exports = {
  i18n: {
    locales: ['vi', 'en'], // Danh sách ngôn ngữ
    defaultLocale: 'vi', // Ngôn ngữ mặc định
    localeDetection: true, // Tự động phát hiện ngôn ngữ từ trình duyệt
  },
};
```
- **Cách hoạt động**:
  - `locales`: Danh sách mã ngôn ngữ được hỗ trợ (ví dụ: `vi` cho tiếng Việt, `en` cho tiếng Anh).
  - `defaultLocale`: Ngôn ngữ được sử dụng khi không có tiền tố ngôn ngữ trong URL.
  - `localeDetection`: Tự động chuyển hướng dựa trên ngôn ngữ trình duyệt của người dùng.
- **Ví dụ**:
  - Với cấu hình trên, Next.js sẽ:
    - Phục vụ `/` và `/vi` cho tiếng Việt (mặc định).
    - Phục vụ `/en` cho tiếng Anh.
  - Trang `/about` sẽ có sẵn dưới dạng `/vi/about` và `/en/about`.
- **Ưu điểm**:
  - Tích hợp sẵn trong Next.js, không cần thư viện bổ sung.
  - Hỗ trợ SEO với các URL riêng cho từng ngôn ngữ.
  - Tự động phát hiện ngôn ngữ người dùng.
- **Nhược điểm**:
  - Cần quản lý nội dung dịch thuật thủ công hoặc với thư viện bổ sung.
  - Không hỗ trợ các tính năng dịch thuật nâng cao (ví dụ: số nhiều, định dạng ngày).
- **Trường hợp sử dụng**: Các ứng dụng cần hỗ trợ nhiều ngôn ngữ, như trang web thương mại điện tử hoặc blog quốc tế.

## 2. Thư viện hỗ trợ: next-i18next, react-intl

### a. next-i18next
- **Mục đích**: Thư viện phổ biến để quản lý nội dung đa ngôn ngữ trong Next.js, tích hợp tốt với i18n của Next.js.
- **Cài đặt**:
  - Cài đặt: `npm install next-i18next`
  - Tạo file cấu hình `next-i18next.config.js`:
```javascript
module.exports = {
  i18n: {
    locales: ['vi', 'en'],
    defaultLocale: 'vi',
  },
};
```
  - Cập nhật `next.config.js`:
```javascript
const { i18n } = require('./next-i18next.config');

/** @type {import('next').NextConfig} */
module.exports = {
  i18n,
};
```
  - Tạo file dịch thuật, ví dụ:
    - `public/locales/vi/common.json`:
```json
{
  "welcome": "Chào mừng đến với ứng dụng của chúng tôi",
  "description": "Đây là một ứng dụng Next.js hỗ trợ đa ngôn ngữ"
}
```
    - `public/locales/en/common.json`:
```json
{
  "welcome": "Welcome to our application",
  "description": "This is a Next.js application with multi-language support"
}
```
- **Sử dụng**:
```javascript
// pages/index.js
import { useTranslation } from 'next-i18next';
import { serverSideTranslations } from 'next-i18next/serverSideTranslations';

export default function Home() {
  const { t } = useTranslation('common');

  return (
    <div>
      <h1>{t('welcome')}</h1>
      <p>{t('description')}</p>
    </div>
  );
}

export async function getStaticProps({ locale }) {
  return {
    props: {
      ...(await serverSideTranslations(locale, ['common'])),
    },
  };
}
```
- **Ưu điểm**:
  - Tích hợp mượt mà với Next.js i18n.
  - Hỗ trợ tải file dịch thuật động.
  - Dễ quản lý với cấu trúc file JSON.
- **Nhược điểm**:
  - Cần cấu hình file dịch thuật riêng cho mỗi ngôn ngữ.
  - Thêm phụ thuộc vào thư viện.
- **Trường hợp sử dụng**: Các dự án cần quản lý dịch thuật tập trung với file JSON.

### b. react-intl
- **Mục đích**: Thư viện mạnh mẽ để xử lý đa ngôn ngữ, hỗ trợ các tính năng nâng cao như định dạng số, ngày, và số nhiều.
- **Cài đặt**:
  - Cài đặt: `npm install react-intl`
- **Sử dụng**:
```javascript
// components/IntlProviderWrapper.js
import { IntlProvider } from 'react-intl';
import { useRouter } from 'next/router';

const messages = {
  vi: {
    welcome: 'Chào mừng đến với ứng dụng của chúng tôi',
    description: 'Đây là một ứng dụng Next.js hỗ trợ đa ngôn ngữ',
  },
  en: {
    welcome: 'Welcome to our application',
    description: 'This is a Next.js application with multi-language support',
  },
};

export default function IntlProviderWrapper({ children }) {
  const { locale } = useRouter();

  return (
    <IntlProvider locale={locale} messages={messages[locale]}>
      {children}
    </IntlProvider>
  );
}
```
```javascript
// pages/_app.js
import IntlProviderWrapper from '../components/IntlProviderWrapper';

export default function MyApp({ Component, pageProps }) {
  return (
    <IntlProviderWrapper>
      <Component {...pageProps} />
    </IntlProviderWrapper>
  );
}
```
```javascript
// pages/index.js
import { useIntl } from 'react-intl';

export default function Home() {
  const { formatMessage } = useIntl();

  return (
    <div>
      <h1>{formatMessage({ id: 'welcome' })}</h1>
      <p>{formatMessage({ id: 'description' })}</p>
    </div>
  );
}
```
- **Ưu điểm**:
  - Hỗ trợ định dạng nâng cao (số, ngày, số nhiều).
  - Phù hợp với các ứng dụng phức tạp cần xử lý ngôn ngữ chi tiết.
- **Nhược điểm**:
  - Cấu hình phức tạp hơn so với next-i18next.
  - Thêm phụ thuộc và có thể làm tăng kích thước bundle.
- **Trường hợp sử dụng**: Các ứng dụng cần định dạng ngôn ngữ phức tạp, như ứng dụng tài chính hoặc quốc tế hóa cao cấp.

## 3. Routing theo ngôn ngữ (/en, /vi, ...)

- **Mục đích**: Tạo các URL riêng cho từng ngôn ngữ (ví dụ: `/en/about`, `/vi/about`) để hỗ trợ SEO và trải nghiệm người dùng.
- **Cách thực hiện**:
  - Sử dụng cấu hình i18n trong `next.config.js` để tự động thêm tiền tố ngôn ngữ vào URL.
  - Tùy chỉnh chuyển đổi ngôn ngữ bằng `next/router` hoặc `<Link>`.
- **Ví dụ chuyển đổi ngôn ngữ**:
```javascript
// components/LanguageSwitcher.js
import { useRouter } from 'next/router';
import Link from 'next/link';

export default function LanguageSwitcher() {
  const router = useRouter();
  const { locale, locales, asPath } = router;

  return (
    <div>
      {locales.map((loc) => (
        <Link key={loc} href={asPath} locale={loc}>
          <a className={locale === loc ? 'active' : ''}>{loc.toUpperCase()}</a>
        </Link>
      ))}
    </div>
  );
}
```
```css
/* styles/LanguageSwitcher.module.css */
a {
  margin-right: 10px;
  text-decoration: none;
}
a.active {
  font-weight: bold;
  color: #0070f3;
}
```
- **Sử dụng với getStaticProps/getServerSideProps**:
```javascript
// pages/[locale]/about.js
import { useTranslation } from 'next-i18next';
import { serverSideTranslations } from 'next-i18next/serverSideTranslations';

export default function About() {
  const { t } = useTranslation('common');

  return (
    <div>
      <h1>{t('about')}</h1>
      <p>{t('about_description')}</p>
    </div>
  );
}

export async function getStaticProps({ locale }) {
  return {
    props: {
      ...(await serverSideTranslations(locale, ['common'])),
    },
  };
}
```
- **File dịch thuật**:
  - `public/locales/vi/common.json`:
```json
{
  "about": "Giới thiệu",
  "about_description": "Đây là trang giới thiệu của chúng tôi"
}
```
  - `public/locales/en/common.json`:
```json
{
  "about": "About",
  "about_description": "This is our about page"
}
```
- **Ưu điểm**:
  - URL thân thiện với SEO (mỗi ngôn ngữ có đường dẫn riêng).
  - Dễ dàng chuyển đổi ngôn ngữ với `<Link>` hoặc `router.push`.
  - Tích hợp tốt với các thư viện như next-i18next.
- **Nhược điểm**:
  - Cần quản lý nội dung dịch thuật cho mỗi ngôn ngữ.
  - Có thể tăng độ phức tạp khi có nhiều ngôn ngữ.
- **Trường hợp sử dụng**: Trang web quốc tế hóa, như thương mại điện tử, blog, hoặc ứng dụng phục vụ nhiều khu vực.