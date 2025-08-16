# VI. State Management trong Next.js

## 1. State trong component (React cơ bản)

- **Mục đích**: Quản lý trạng thái (state) cục bộ trong một component React, sử dụng hook `useState` hoặc `useReducer` để xử lý dữ liệu nội tại.
- **Cách hoạt động**:
  - Sử dụng `useState` cho trạng thái đơn giản.
  - Sử dụng `useReducer` cho trạng thái phức tạp hoặc có nhiều hành động liên quan.
- **Ví dụ với useState**:
```javascript
// components/Counter.js
import { useState } from 'react';

export default function Counter() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>Số lần nhấn: {count}</p>
      <button onClick={() => setCount(count + 1)}>Tăng</button>
    </div>
  );
}
```
- **Ví dụ với useReducer**:
```javascript
// components/CounterReducer.js
import { useReducer } from 'react';

const initialState = { count: 0 };

function reducer(state, action) {
  switch (action.type) {
    case 'increment':
      return { count: state.count + 1 };
    case 'decrement':
      return { count: state.count - 1 };
    default:
      return state;
  }
}

export default function CounterReducer() {
  const [state, dispatch] = useReducer(reducer, initialState);

  return (
    <div>
      <p>Số lần nhấn: {state.count}</p>
      <button onClick={() => dispatch({ type: 'increment' })}>Tăng</button>
      <button onClick={() => dispatch({ type: 'decrement' })}>Giảm</button>
    </div>
  );
}
```
- **Ưu điểm**:
  - Đơn giản, dễ sử dụng cho trạng thái cục bộ.
  - Không cần thêm thư viện bên ngoài.
  - Tích hợp tốt với React và Next.js.
- **Nhược điểm**:
  - Không phù hợp cho trạng thái toàn cục (global state) hoặc chia sẻ giữa nhiều component.
  - Có thể dẫn đến code lặp lại nếu trạng thái được sử dụng ở nhiều nơi.
- **Trường hợp sử dụng**: Quản lý trạng thái cục bộ như form input, toggle switch, hoặc các UI component độc lập.

## 2. Context API trong Next.js

- **Mục đích**: Cung cấp cách chia sẻ trạng thái toàn cục giữa các component mà không cần truyền props qua nhiều cấp.
- **Cách hoạt động**:
  - Tạo một Context với `createContext`.
  - Sử dụng `Provider` để cung cấp giá trị trạng thái.
  - Sử dụng `useContext` để truy cập trạng thái trong các component con.
- **Ví dụ**:
```javascript
// context/ThemeContext.js
import { createContext, useContext, useState } from 'react';

const ThemeContext = createContext();

export function ThemeProvider({ children }) {
  const [theme, setTheme] = useState('light');

  const toggleTheme = () => {
    setTheme(theme === 'light' ? 'dark' : 'light');
  };

  return (
    <ThemeContext.Provider value={{ theme, toggleTheme }}>
      {children}
    </ThemeContext.Provider>
  );
}

export function useTheme() {
  return useContext(ThemeContext);
}
```
```javascript
// pages/_app.js
import { ThemeProvider } from '../context/ThemeContext';

export default function MyApp({ Component, pageProps }) {
  return (
    <ThemeProvider>
      <Component {...pageProps} />
    </ThemeProvider>
  );
}
```
```javascript
// components/ThemeToggle.js
import { useTheme } from '../context/ThemeContext';

export default function ThemeToggle() {
  const { theme, toggleTheme } = useTheme();

  return (
    <div>
      <p>Chủ đề hiện tại: {theme}</p>
      <button onClick={toggleTheme}>Đổi chủ đề</button>
    </div>
  );
}
```
- **Ưu điểm**:
  - Tích hợp sẵn trong React, không cần thêm thư viện.
  - Phù hợp cho trạng thái toàn cục đơn giản, như chủ đề, ngôn ngữ, hoặc dữ liệu người dùng.
  - Dễ sử dụng với Next.js.
- **Nhược điểm**:
  - Hiệu suất giảm khi trạng thái phức tạp hoặc có nhiều cập nhật (vì Context re-render tất cả consumer).
  - Không phù hợp cho ứng dụng lớn với trạng thái phức tạp.
- **Trường hợp sử dụng**: Quản lý trạng thái toàn cục đơn giản, như cài đặt người dùng, chủ đề, hoặc thông tin xác thực.

## 3. Redux Toolkit / Zustand (ứng dụng nâng cao)

### a. Redux Toolkit
- **Mục đích**: Quản lý trạng thái toàn cục phức tạp với Redux, được tối ưu hóa bởi Redux Toolkit để giảm boilerplate code.
- **Cài đặt**: `npm install @reduxjs/toolkit react-redux`
- **Cách sử dụng**:
  - Tạo slice để định nghĩa state và reducer.
  - Cấu hình store và cung cấp cho ứng dụng qua `Provider`.
  - Sử dụng `useSelector` và `useDispatch` để truy cập và cập nhật state.
- **Ví dụ**:
```javascript
// store/counterSlice.js
import { createSlice } from '@reduxjs/toolkit';

const counterSlice = createSlice({
  name: 'counter',
  initialState: { value: 0 },
  reducers: {
    increment: (state) => {
      state.value += 1;
    },
    decrement: (state) => {
      state.value -= 1;
    },
  },
});

export const { increment, decrement } = counterSlice.actions;
export default counterSlice.reducer;
```
```javascript
// store/index.js
import { configureStore } from '@reduxjs/toolkit';
import counterReducer from './counterSlice';

export const store = configureStore({
  reducer: {
    counter: counterReducer,
  },
});
```
```javascript
// pages/_app.js
import { Provider } from 'react-redux';
import { store } from '../store';

export default function MyApp({ Component, pageProps }) {
  return (
    <Provider store={store}>
      <Component {...pageProps} />
    </Provider>
  );
}
```
```javascript
// components/Counter.js
import { useSelector, useDispatch } from 'react-redux';
import { increment, decrement } from '../store/counterSlice';

export default function Counter() {
  const count = useSelector((state) => state.counter.value);
  const dispatch = useDispatch();

  return (
    <div>
      <p>Số lần nhấn: {count}</p>
      <button onClick={() => dispatch(increment())}>Tăng</button>
      <button onClick={() => dispatch(decrement())}>Giảm</button>
    </div>
  );
}
```
- **Ưu điểm**:
  - Cung cấp công cụ mạnh mẽ để quản lý trạng thái phức tạp.
  - Redux Toolkit giảm boilerplate code và tích hợp các best practices.
  - Có DevTools để debug dễ dàng.
- **Nhược điểm**:
  - Thêm phụ thuộc vào thư viện.
  - Cần học cách sử dụng (slice, reducer, action).
- **Trường hợp sử dụng**: Các ứng dụng lớn với trạng thái phức tạp, như ứng dụng thương mại điện tử hoặc quản lý dữ liệu thời gian thực.

### b. Zustand
- **Mục đích**: Thư viện quản lý trạng thái nhẹ, đơn giản, không cần boilerplate như Redux.
- **Cài đặt**: `npm install zustand`
- **Cách sử dụng**:
  - Tạo store với `create` và sử dụng hook để truy cập state.
- **Ví dụ**:
```javascript
// store/counterStore.js
import { create } from 'zustand';

const useCounterStore = create((set) => ({
  count: 0,
  increment: () => set((state) => ({ count: state.count + 1 })),
  decrement: () => set((state) => ({ count: state.count - 1 })),
}));

export default useCounterStore;
```
```javascript
// components/Counter.js
import useCounterStore from '../store/counterStore';

export default function Counter() {
  const { count, increment, decrement } = useCounterStore();

  return (
    <div>
      <p>Số lần nhấn: {count}</p>
      <button onClick={increment}>Tăng</button>
      <button onClick={decrement}>Giảm</button>
    </div>
  );
}
```
- **Ưu điểm**:
  - Nhẹ, dễ học, và không cần boilerplate.
  - Hiệu suất tốt, chỉ re-render component liên quan.
  - Tích hợp tốt với Next.js.
- **Nhược điểm**:
  - Ít tính năng hơn Redux Toolkit (như DevTools tích hợp sẵn).
  - Không phù hợp cho các ứng dụng cực kỳ phức tạp.
- **Trường hợp sử dụng**: Ứng dụng cần quản lý trạng thái toàn cục đơn giản, muốn giảm phụ thuộc và boilerplate.

## 4. Kết hợp với SWR / React Query để quản lý state từ server

- **Mục đích**: Sử dụng SWR hoặc React Query để quản lý trạng thái lấy từ server (server state), kết hợp với trạng thái cục bộ hoặc toàn cục.
- **Cách hoạt động**:
  - SWR và React Query chuyên về quản lý dữ liệu từ API (server state), như caching, refetching, và xử lý trạng thái tải/lỗi.
  - Kết hợp với Context API, Redux Toolkit, hoặc Zustand để quản lý trạng thái cục bộ hoặc toàn cục.

### a. SWR
- **Cách sử dụng**:
  - Sử dụng hook `useSWR` để lấy dữ liệu từ API và quản lý trạng thái.
- **Ví dụ**:
```javascript
// components/UserList.js
import useSWR from 'swr';

const fetcher = (url) => fetch(url).then((res) => res.json());

export default function UserList() {
  const { data, error } = useSWR('/api/users', fetcher);

  if (error) return <div>Lỗi khi tải dữ liệu</div>;
  if (!data) return <div>Đang tải...</div>;

  return (
    <ul>
      {data.map((user) => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
}
```
- **Kết hợp với Zustand**:
```javascript
// store/userStore.js
import { create } from 'zustand';

const useUserStore = create((set) => ({
  users: [],
  setUsers: (users) => set({ users }),
}));

export default useUserStore;
```
```javascript
// components/UserList.js
import useSWR from 'swr';
import useUserStore from '../store/userStore';

const fetcher = (url) => fetch(url).then((res) => res.json());

export default function UserList() {
  const { setUsers } = useUserStore();
  const { data, error } = useSWR('/api/users', fetcher, {
    onSuccess: (data) => setUsers(data), // Lưu dữ liệu vào store
  });

  if (error) return <div>Lỗi khi tải dữ liệu</div>;
  if (!data) return <div>Đang tải...</div>;

  return (
    <ul>
      {data.map((user) => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
}
```

### b. React Query
- **Cách sử dụng**:
  - Sử dụng hook `useQuery` để lấy dữ liệu và quản lý trạng thái.
- **Cài đặt**: `npm install @tanstack/react-query`
- **Ví dụ**:
```javascript
// components/UserList.js
import { useQuery } from '@tanstack/react-query';

export default function UserList() {
  const { data, isLoading, error } = useQuery({
    queryKey: ['users'],
    queryFn: () => fetch('/api/users').then((res) => res.json()),
  });

  if (isLoading) return <div>Đang tải...</div>;
  if (error) return <div>Lỗi: {error.message}</div>;

  return (
    <ul>
      {data.map((user) => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
}
```
- **Kết hợp với Redux Toolkit**:
```javascript
// store/userSlice.js
import { createSlice } from '@reduxjs/toolkit';

const userSlice = createSlice({
  name: 'users',
  initialState: { users: [] },
  reducers: {
    setUsers: (state, action) => {
      state.users = action.payload;
    },
  },
});

export const { setUsers } = userSlice.actions;
export default userSlice.reducer;
```
```javascript
// components/UserList.js
import { useQuery } from '@tanstack/react-query';
import { useDispatch, useSelector } from 'react-redux';
import { setUsers } from '../store/userSlice';

export default function UserList() {
  const dispatch = useDispatch();
  const users = useSelector((state) => state.users.users);

  const { isLoading, error } = useQuery({
    queryKey: ['users'],
    queryFn: () => fetch('/api/users').then((res) => res.json()),
    onSuccess: (data) => dispatch(setUsers(data)), // Lưu dữ liệu vào Redux store
  });

  if (isLoading) return <div>Đang tải...</div>;
  if (error) return <div>Lỗi: {error.message}</div>;

  return (
    <ul>
      {users.map((user) => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
}
```
- **Ưu điểm**:
  - SWR/React Query tự động xử lý caching, refetching, và trạng thái tải/lỗi.
  - Giảm boilerplate code khi lấy dữ liệu từ server.
  - Kết hợp tốt với các công cụ quản lý trạng thái khác (Context, Redux, Zustand).
- **Nhược điểm**:
  - Thêm phụ thuộc vào thư viện.
  - Cần cấu hình thêm nếu kết hợp với store toàn cục.
- **Trường hợp sử dụng**:
  - Quản lý dữ liệu từ server, như danh sách người dùng, sản phẩm, hoặc bài viết.
  - Kết hợp với trạng thái toàn cục để đồng bộ dữ liệu giữa client và server.