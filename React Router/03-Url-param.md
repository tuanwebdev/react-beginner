
## 1. URL Params

Dùng để truyền dữ liệu động trực tiếp trên URL.

Ví dụ:

```bash
/users/123
```

### Khai báo route

```jsx
<Route path="/users/:id" element={<UserProfile />} />
```

### Lấy dữ liệu bằng `useParams`

```jsx
import { useParams } from 'react-router-dom';

function UserProfile() {
  const { id } = useParams();

  return <div>User ID: {id}</div>;
}
```

### Kết quả

```js
{id: "123"}
```

---

# 2. Query String

Dùng cho filter, search, sort, pagination...

Ví dụ:

```bash
/search?query=react&sort=asc
```

> Không cần khai báo trong route.

---

## Dùng `useSearchParams`

```jsx
import { useSearchParams } from 'react-router-dom';

function SearchPage() {
  const [searchParams, setSearchParams] = useSearchParams();

  // đọc dữ liệu
  const query = searchParams.get('query');

  // cập nhật query string
  const handleUpdate = () => {
    setSearchParams({
      query: 'new-search',
      sort: 'desc',
    });
  };

  return (
    <>
      <p>{query}</p>

      <button onClick={handleUpdate}>
        Update
      </button>
    </>
  );
}
```

---

# 3. Navigate kèm Query String

## Cách 1 — Template String

```jsx
navigate(`/results?query=react`);
```

---

## Cách 2 — `createSearchParams`

Phù hợp khi query nhiều field.

```jsx
import {
  useNavigate,
  createSearchParams,
} from 'react-router-dom';

const navigate = useNavigate();

navigate({
  pathname: '/results',
  search: `?${createSearchParams({
    query: 'react',
    sort: 'asc',
  })}`,
});
```

---

# 4. So sánh nhanh

| URL Params           | Query String                |
| -------------------- | --------------------------- |
| `/user/123`          | `/user?id=123`              |
| Cần khai báo route   | Không cần                   |
| `useParams()`        | `useSearchParams()`         |
| Dùng cho ID/resource | Dùng cho filter/search/sort |

---

# 5. Ghi nhớ nhanh

## URL Params

```jsx
/users/:id
```

```jsx
const { id } = useParams();
```

---

## Query String

```bash
/search?q=react
```

```jsx
const [searchParams] = useSearchParams();

searchParams.get('q');
```

---

# 6. Khi nào dùng?

| Trường hợp            | Nên dùng     |
| --------------------- | ------------ |
| Chi tiết user/product | URL Params   |
| Search/filter/sort    | Query String |
| Pagination            | Query String |
| SEO-friendly URL      | URL Params   |
