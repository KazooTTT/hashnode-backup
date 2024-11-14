---
title: "search params 请求参数的获取与更新"
datePublished: Thu Nov 14 2024 02:01:18 GMT+0000 (Coordinated Universal Time)
cuid: cm3gnyz4b000009mc37bsc9vg
slug: search-params
tags: react-router, frontend-development, windowlocation

---

## react router `useSearchParams`

[useSearchParams | React Router](https://reactrouter.com/en/main/hooks/use-search-params#usesearchparams)

```ts
interface URLSearchParams {
    /** Appends a specified key/value pair as a new search parameter. */
    append(name: string, value: string): void;
    /** Deletes the given search parameter, and its associated value, from the list of all search parameters. */
    delete(name: string): void;
    /** Returns the first value associated to the given search parameter. */
    get(name: string): string | null;
    /** Returns all the values association with a given search parameter. */
    getAll(name: string): string[];
    /** Returns a Boolean indicating if such a search parameter exists. */
    has(name: string): boolean;
    /** Sets the value associated to a given search parameter to the given value. If there were several values, delete the others. */
    set(name: string, value: string): void;
    sort(): void;
    /** Returns a string containing a query string suitable for use in a URL. Does not include the question mark. */
    toString(): string;
    forEach(callbackfn: (value: string, key: string, parent: URLSearchParams) => void, thisArg?: any): void;
}
```

它提供了内建的 API，允许直接获取查询参数的值，比如 `.get()`, `.set()`, `.append()` 等。

```ts
  const [queryParams, setQueryParams] = useSearchParams();
  console.log('%c Line:52 🍿 queryParams', 'color:#33a5ff', queryParams.get('medicalRecordID'));
```

## qs + window.location

[GitHub - ljharb/qs: A querystring parser with nesting support](https://github.com/ljharb/qs)

![image.png](https://pictures.kazoottt.top/2024/11/20241113-9c3c37d82dd684dc8ca2b75cfb16784e.png align="left")

[使用window.location.search](http://使用window.location.search)获取到请求参数对应的字符串（需要注意的是：字符串是带有?的）

然后使用qs.parse方法来解析查询字符串

案例：[localhost](http://localhost)?medicalRecordID=1

错误使用:

```ts
const getQueryParam = (): QueryParams => {
  // use qs to parse the query params
  const queryParams: QueryParams = qs.parse(window.location.search);
  return queryParams;
};
```

![](https://pictures.kazoottt.top/2024/11/20241113-157c9570908a6b1f584ae28db3eebf1d.png align="left")

正确使用：

```ts
const getQueryParam = (): QueryParams => {
  // use qs to parse the query params
  const queryParams: QueryParams = qs.parse(window.location.search.slice(1));
  return queryParams;
};
```

![image.png](https://pictures.kazoottt.top/2024/11/20241113-48ad512e7639c8027216269380b7cacf.png align="left")