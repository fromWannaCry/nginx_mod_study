# Nginx Mod开发学习

## 零件部分

### 使用到的头文件

```c++
#include <nginx.h>
extern "C" {
	#include <ngx_http.h>
}
```

### 常用的类型

```c++
size_t
u_char
off_t
```

### NGX类型

```c
// 定义在 core/ngx_config.h
ngx_int_t
ngx_uing_t
ngx_flag_t
// 定义在 os/unix/ngx_time.h，毫秒的整数类型
typedef ngx_rbtree_key_t ngx_msec_t;
typedef ngx_rbtree_key_int_t ngx_msec_int_t;
```

### 无效值

`Nginx`使用`-1`表示无效值，但相同的`-1`有很多不同的表示方式，例如：

```c
// 定义于core/ngx_conf_file.h
#define NGX_CONF_UNSET       -1
#define NGX_CONF_UNSET_UINT  (ngx_uint_t) -1
#define NGX_CONF_UNSET_PTR   (void *) -1
#define NGX_CONF_UNSET_SIZE  (size_t) -1
#define NGX_CONF_UNSET_MSEC  (ngx_msec_t) -1
```

同时就有了**初始化**和**条件赋值**的函数

```c
// 定义于core/ngx_conf_file.h
#define ngx_conf_init_value(conf, default)                                   \
    if (conf == NGX_CONF_UNSET) {                                            \
        conf = default;                                                      \
    }
#define ngx_conf_init_ptr_value(conf, default)                               \
    if (conf == NGX_CONF_UNSET_PTR) {                                        \
        conf = default;                                                      \
    }
#define ngx_conf_merge_value(conf, prev, default)                            \
    if (conf == NGX_CONF_UNSET) {                                            \
        conf = (prev == NGX_CONF_UNSET) ? default : prev;                    \
    }
```

### 错误处理

`Nginx`定义了`7`个常用错误码，类型是`ngx_int_t`

```c
// 定义在 core/ngx_core.h
#define  NGX_OK          0
#define  NGX_ERROR      -1
#define  NGX_AGAIN      -2
#define  NGX_BUSY       -3
#define  NGX_DONE       -4
#define  NGX_DECLINED   -5
#define  NGX_ABORT      -6
```

### 内存池

`Nginx`使用内存池来管理内存，申请时应使用系列函数申请

#### 结构定义

```c
// 结构定义
// 定义在ngx_core.h
typedef struct ngx_pool_s ngx_pool_t;

// 定义在ngx_palloc.h
struct ngx_pool_s {
    ngx_pool_data_t       d;
    size_t                max;
    ngx_pool_t           *current;
    ngx_chain_t          *chain;
    ngx_pool_large_t     *large;
    ngx_pool_cleanup_t   *cleanup;		// 析构时的清理动作
    ngx_log_t            *log;			// 关联的日志对象
};
```

#### 操作函数

相对应的，就有**申请内存**和**释放内存**用的操作函数

```c
// 操作函数
//
void *ngx_palloc(ngx_pool_t *pool, size_t size);
void *ngx_pnalloc(ngx_pool_t *pool, size_t size);
void *ngx_pcalloc(ngx_pool_t *pool, size_t size);
void *ngx_pmemalign(ngx_pool_t *pool, size_t size, size_t alignment);
ngx_int_t ngx_pfree(ngx_pool_t *pool, void *p);
```

`ngx_palloc()`使用了内存对齐，速度快，但有部分内存浪费
`ngx_pnalloc()`没有使用内存对齐
`ngx_pcalloc()`不仅内部调用`ngx_palloc()`，还有**清零**操作

#### 清理机制

























