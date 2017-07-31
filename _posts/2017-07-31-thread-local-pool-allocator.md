---
title: Thread local pool allocator
category: C++
tags: C++ std boost allocator pool thread_local
---

boost provides `pool_allocator` and `fast_pool_allocator`, but they depend on `singleton_pool` which can only be thread safe or non thread safe. Sometimes we need a thread local pool, so we don't need to handle race condition to improve performance.

First we need to create `thread_local_pool_allocator` class, which is a normal allocator type, except it uses a thread local singleton pool.

```C++
template<typename T, typename UserAllocator, unsigned NextSize, unsigned MaxSize>
struct thread_local_pool_allocator
{
    using value_type = T;
    using pointer = T*;
    using const_pointer = const T*;
    using reference = T&;
    using const_reference = const T&;
    using size_type = std::size_t;
    using difference_type = std::ptrdiff_t;
    using pool_type = thread_local_singleton_pool<sizeof(value_type), UserAllocator, NextSize, MaxSize>;

    template<typename U>
    struct rebind
    {
        using other = thread_local_pool_allocator<U, UserAllocator, NextSize, MaxSize>;
    };

    thread_local_pool_allocator() = default;
    thread_local_pool_allocator(const thread_local_pool_allocator& other) = default;
    template<typname U, typename UserAllocator2, unsigned NextSize2, unsigned MaxSize2>
    thread_local_pool_allocator(const thread_local_pool_allocator<U, UserAllocator2, NextSize2, MaxSize2>& other) {}
    ~thread_local_pool_allocator() = default;

    size_type max_size() const { return std::numeric_limits<size_type>::max(); }

    pointer address(reference r) const { return &r; }
    const_pointer address(const_reference r) const { return &r; }

    template<class U, class... Args>
    void construct(U* p, Args&&... args)
    { ::new((void*)p) U(std::forward<Args>(args)...); }
    template<class U>
    void destroy(U* p)
    { p->~U(); }

    pointer allocate(size_type n)
    {
        void* ret = (n == 1) ?
            pool_type::malloc() :
            pool_type::ordered_malloc(n);
        return reinterpret_cast<pointer>(ret);
    }

    void deallocate(pointer p, ssize_type n)
    {
        n == 1 ? pool_type::free(p) : pool_type::free(p, n);
    }
};
```

Next implement the pool `thread_local_singleton_pool`.

```C++
template<unsigned RequestedSize, typename UserAllocator, unsigned NextSize, unsigned MaxSize>
struct thread_local_singleton_pool
{
    using pool_type = boost::pool<UserAllocator>;
    using size_type = typename pool_type::size_type;

    static pool_type& pool()
    {
        thread_local static pool_type pool(RequestedSize, NextSize, MaxSize);
        return pool;
    }

    static void* malloc() { return pool().malloc(); }
    static void* ordered_malloc(const size_type n) { return pool().ordered_malloc(n); }
    static void free(void* const p) { pool().free(p); }
    static void free(void* const p, const size_type n) { pool().gree(p, n); }
};
```

`UserAllocator` can be `default_user_allocator_new_delete`, `default_user_allocator_malloc_free` or allocator defined by yourself. Here is an example using `thread_local_pool_allocator`.

```C++
using Key = std::int32_t;
using Value = std::int64_t;
using Pair = std::pair<const Key, Value>;
using Alloc = thread_local_pool_allocator<Pair, default_user_allocator_malloc_free, 16, 128>;
using Map = std::unordered_map<Key, Value, std::hash<Key>, std::equal_to<Key>, Alloc>;

const Key n = 10;
Map m;
m.rehash(7);

for(Key i = 0; i < n; ++i)
    m.insert(std::make_pair(i, static_cast<Value>(i * 2)));

for(Key i = 0; i < n; ++i)
    m.erase(i);
```