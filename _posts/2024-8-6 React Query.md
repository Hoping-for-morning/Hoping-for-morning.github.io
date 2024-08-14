---
title: 在 React useEffect 中发送异步请求
date: 2024-08-6
categories: [Front-end]
tags: [frontend, React]  
---

# 在 React useEffect 中发送异步请求

## 需求

1. 编辑器停止编辑时，自动发送异步请求
    `注意` 在响应前不要发送下一次请求，否则会发生多次请求
2. 设置响应延时，一段时间后才发送异步请求
3. 得到响应后，将内容插入到文章中
    `注意` 有可能在得到响应前依赖就发生变更了，相当于得到的响应是过时的应该被抛弃掉



## React 中在 useEffect 里请求数据

参考文章 [为什么你不应该在 React 中直接使用 useEffect 从 API 获取数据](https://blog.skk.moe/post/why-you-should-not-fetch-data-directly-in-use-effect/)

> 网络请求不属于渲染，而是渲染的副作用，React 提供了一个专门的 Hook `useEffect` 用来处理渲染的副作用，于是想到了在 `useEffect` 中进行异步请求，

- isLoading
- error 上报
- 封装一个 useFetch 的 hook
- 清除副作用，处理 race condition
- 缓存网络请求
- 缓存刷新
- 请求合并

```tsx
import React, { useEffect, useState } from 'react';

const MyComponent = () => {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);
  const isMounted = useRef(true);

  useEffect(() => {
    isMounted.current = true;
    let timerId;

    const fetchData = async () => {
      setLoading(true);
      try {
        const response = await fetch('https://api.example.com/data');
        if (!response.ok) {
          throw new Error('Network response was not ok');
        }
        const result = await response.json();
        if (isMounted.current) {
          setData(result);
        }
      } catch (error) {
        setError(error);
      } finally {
        setLoading(false);
        // 设置下一次请求的定时器
        timerId = setTimeout(fetchData, 5000); // 例如每5秒发送一次请求
      }
    };

    fetchData();

    // 清理函数
    return () => {
      isMounted.current = false;
      if (timerId) {
        clearTimeout(timerId);
      }
    };
  }, []); // 空数组表示该 effect 只在组件挂载和卸载时执行一次

  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;

  return (
    <div>
      <h1>Data</h1>
      <pre>{JSON.stringify(data, null, 2)}</pre>
    </div>
  );
};

export default MyComponent;
```

