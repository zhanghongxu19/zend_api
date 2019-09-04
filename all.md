# 参数相关

```c
int zend_parse_parameters(int num_args TSRMLS_DC, char *type_spec, ...)

```

# 返回值相关

```c
RETURN_NULL()
RETURN_BOOL(b) b: 0 => FALSE, non-0 => TRUE
RETURN_TRUE
RETURN_FALSE
RETURN_LONG(l) l: Integer value
RETURN_DOUBLE(d) d: Floating point value
RETURN_STRING(str, dup) str: char* string value dup: 0/1 flag, duplicate string?
RETURN_STRINGL(str, len, dup) len: Predetermined string length
RETURN_EMPTY_STRING()
```

# 字符串相关

```c

void zend_str_tolower(char *str, size_t len);  // 转换字符串 char * 为小写
zend_string* zend_string_tolower(zend_string *str);  // 转换字符串 zend_string * 为小写
zend_string* php_string_tolower(zend_string *str);  // 功能跟上面一样，是 strtolower 的原型
zend_string* php_string_toupper(zend_string *str);  // strtoupper 的原型

// 格式化成 zend_string *, 使用完记得 zend_string_release
zend_string *strpprintf(size_t max_len, const char *format, ...);
// 格式化成 char *, 使用完记得 efree
size_t spprintf( char **pbuf, size_t max_len, const char *format, ...);

smart_str_*  // 待分析

convert_to_cstring
convert_to_string

zend_string *zval_get_string(zval *val);  // 从 zval 转换为 zend_string 并 COPY 一份


char *ZSTR_VAL(zend_string *s)
size_t ZSTR_LEN(zend_string *s)
zend_ulong ZSTR_H(zend_string *s)
zend_ulong ZSTR_HASH(zend_string *s) 
// 是函数 zend_string_hash_val 的宏
// 与 ZSTR_H 的宏功能差不多，多了个判断操作，如果zand_string的h字段为空，会重新计算并设置
ZSTR_IS_INTERNED


zend_ulong zend_string_hash_val(zend_string *s)
void zend_string_forget_hash_val(zend_string *s)
uint32_t zend_string_refcount(const zend_string *s)
uint32_t zend_string_addref(zend_string *s)
uint32_t zend_string_delref(zend_string *s)
zend_string *zend_string_alloc(size_t len, int persistent)
zend_string *zend_string_safe_alloc(size_t n, size_t m, size_t l, int persistent)
zend_string *zend_string_init(const char *str, size_t len, int persistent)
zend_string *zend_string_copy(zend_string *s)
zend_string *zend_string_dup(zend_string *s, int persistent)
zend_string *zend_string_realloc(zend_string *s, size_t len, int persistent)
zend_string *zend_string_extend(zend_string *s, size_t len, int persistent)
zend_string *zend_string_truncate(zend_string *s, size_t len, int persistent)
zend_string *zend_string_safe_realloc(zend_string *s, size_t n, size_t m, size_t l, int persistent)
void zend_string_free(zend_string *s)
void zend_string_release(zend_string *s)
zend_bool zend_string_equals(zend_string *s1, zend_string *s2)
```

# 数组相关

## 关联数组

```c
int add_assoc_long(zval *arg, char *key, long n)
add_assoc_null(zval *arg, char *key)
add_assoc_bool(zval *arg, char *key, int b)
add_assoc_resource(zval *arg, char *key, int r)
add_assoc_double(zval *arg, char *key, double d)
add_assoc_string(zval *arg, char *key, char *str, int dup)
int add_assoc_stringl(zval *arg, char *key, char *str, uint len, int dup);
int add_assoc_zval(zval *arg, char *key, zval *value)
```

## 索引数组

```c

// 向数组中指定的数字索引增加指定类型的值
int add_index_long(zval *arg, ulong idx, long n)
int add_index_null(zval *arg, ulong idx)  // $arr[1] = NULL;
int add_index_bool(zval *arg, ulong idx, int b)
int add_index_resource(zval *arg, ulong idx, int r)
int add_index_double(zval *arg, ulong idx, double d)
int add_index_string(zval *arg, ulong idx, const char *str, int duplicate)
int add_index_stringl(zval *arg, ulong idx, const char *str, uint length, int duplicate)
int add_index_zval(zval *arg, ulong index, zval *value)

// 向数字索引的数组增加指定类型的值 
int add_next_index_long(zval *arg, long n)
int add_next_index_null(zval *arg)   // $arr[] = NULL;
int add_next_index_bool(zval *arg, int b)
int add_next_index_resource(zval *arg, int r)
int add_next_index_double(zval *arg, double d)
int add_next_index_string(zval *arg, const char *str, int duplicate)
int add_next_index_stringl(zval *arg, const char *str, uint length, int duplicate)
int add_next_index_zval(zval *arg, zval *value)
```

## init

```c
array_init(zval *val)    // 初始化一个长度为0的数组
array_init_size(zval *val, uint32_t size)   // 初始化一个长度为size的数组
```

## HashTable操作

```
/*
 * HashTable Data Layout
 * =====================
 *
 *                 +=============================+
 *                 | HT_HASH(ht, ht->nTableMask) |
 *                 | ...                         |
 *                 | HT_HASH(ht, -1)             |
 *                 +-----------------------------+
 * ht->arData ---> | Bucket[0]                   |
 *                 | ...                         |
 *                 | Bucket[ht->nTableSize-1]    |
 *                 +=============================+
 */
```

### 宏

```c
HT_HASH(HashTable *ht, uint32_t nTableMask)   // ((uint32_t*)((ht)->arData))[(int32_t)(nTableMask)]
HT_HASH_SIZE(uint32_t nTableMask)   // (((size_t)(uint32_t)-(int32_t)(nTableMask)) * sizeof(uint32_t))
HT_DATA_SIZE
HT_SIZE
HT_SIZE_EX
HT_HASH_RESET(HashTable *ht)   // 将 nTableMask 未全部置为 -1
HT_HASH_RESET_PACKED(HashTable *ht)   // packed array 重置，只是将 nTableMask 的 -1 和 -2 位重置为 -1
HT_HASH_TO_BUCKET
HT_HASH_TO_BUCKET_EX(data)   // ((Bucket*)((char*)(data) + (idx)))
HT_SET_DATA_ADDR
HT_GET_DATA_ADDR
HT_IDX_TO_HASH
HT_HASH_TO_IDX


// 判断是否是 packed array
ht->u.flags & HASH_FLAG_PACKED

```

### 初始化和销毁

```c
int zend_hash_init(HashTable *ht,uint nSize, hash_func_t pHashFunction, dtor_func_t pDestructor, zend_bool persistent);
void zend_hash_destroy(HashTable *ht)
void zend_hash_clean(HashTable *ht) Removes all elements from the HashTable
void zend_hash_real_init(HashTable *ht, zend_bool packed)  // 与 zend_hash_init 类似，分配数组，但是这个函数还分配了内存
```

### 增加/修改

```c
int zend_hash_update(HashTable *ht, const char *arKey, uint nKeyLength, void *pData, uint nDataSize, void **pDest)
int zend_hash_add(HashTable *ht, const char *arKey, uint nKeyLength, void *pData, uint nDataSize, void **pDest)
ulong zend_get_hash_value(char *arKey, uint nKeyLength)
int zend_hash_num_elements(HashTable *ht) // 获取数组的个数 
int zend_hash_quick_add(HashTable *ht, const char *arKey, uint nKeyLength, ulong h, void *pData, uint nDataSize, void **pDest)
int zend_hash_quick_update(HashTable *ht, const char *arKey, uint nKeyLength, ulong h, void *pData, uint nDataSize, void **pDest)
int zend_hash_index_update(HashTable *ht, ulong h, void *pData, uint nDataSize, void **pDest)
int zend_hash_next_index_insert(HashTable *ht, void *pData, uint nDataSize, void **pDest)
int zend_hash_add_empty_element(HashTable *ht, const char *arKey, uint nKeyLength);
void zend_hash_graceful_destroy(HashTable *ht)
void zend_hash_graceful_reverse_destroy(HashTable *ht)
void zend_hash_apply(HashTable *ht, apply_func_t apply_func TSRMLS_DC)
void zend_hash_apply_with_argument(HashTable *ht, apply_func_arg_t apply_func, void * TSRMLS_DC)
void zend_hash_apply_with_arguments(HashTable *ht TSRMLS_DC, apply_func_args_t apply_func, int, ...)
void zend_hash_reverse_apply(HashTable *ht, apply_func_t apply_func TSRMLS_DC);
```

### 删除

```c
int zend_hash_find(const HashTable *ht, const char *arKey, uint nKeyLength, void **pData)
int zend_hash_index_find(const HashTable *ht, ulong h, void **pData)
int zend_hash_quick_find(const HashTable *ht, const char *arKey, uint nKeyLength, ulong h, void **pData);
```
### 查找

```c
int zend_hash_find(const HashTable *ht, const char *arKey, uint nKeyLength, void **pData)
int zend_hash_index_find(const HashTable *ht, ulong h, void **pData)
int zend_hash_quick_find(const HashTable *ht, const char *arKey, uint nKeyLength, ulong h, void **pData);
```

### 其他

```c
int zend_hash_exists(const HashTable *ht, const char *arKey, uint nKeyLength)
int zend_hash_quick_exists(const HashTable *ht, const char *arKey, uint nKeyLength, ulong h)
int zend_hash_index_exists(const HashTable *ht, ulong h)
ulong zend_hash_next_free_element(const HashTable *ht)
```

### traversing

```c
int zend_hash_has_more_elements(ht)
int zend_hash_move_forward(ht)
int zend_hash_move_backwards(ht)
int zend_hash_get_current_key(ht, str_index, num_index, duplicate)
int zend_hash_get_current_key_type(ht)
int zend_hash_get_current_data(ht, pData)
void zend_hash_internal_pointer_reset(ht)
void zend_hash_internal_pointer_end(ht)
int zend_hash_update_current_key(ht, key_type, str_index, str_length, num_index)
```

### Copying, merging and sorting

```c
void zend_hash_copy(HashTable *target, HashTable *source, copy_ctor_func_t pCopyConstructor, void *tmp, uint size)
void _zend_hash_merge(HashTable *target, HashTable *source, copy_ctor_func_t pCopyConstructor, void *tmp, uint size, int overwrite ZEND_FILE_LINE_DC)
void zend_hash_merge_ex(HashTable *target, HashTable *source, copy_ctor_func_t pCopyConstructor, uint size, merge_checker_func_t pMergeSource, void *pParam)
int zend_hash_sort(HashTable *ht, sort_func_t sort_func, compare_func_t compare_func, int renumber TSRMLS_DC)
int zend_hash_compare(HashTable *ht1, HashTable *ht2, compare_func_t compar, zend_bool ordered TSRMLS_DC)
int zend_hash_minmax(const HashTable *ht, compare_func_t compar, int flag, void **pData TSRMLS_DC);
```

### debug

```c
void zend_hash_display_pListTail(const HashTable *ht)
void zend_hash_display(const HashTable *ht)
```

### 遍历

```c
ZEND_HASH_FOREACH_BUCKET(HashTable *ht, Bucket *bucket)
ZEND_HASH_FOREACH_VAL(HashTable *ht, zval *val)
ZEND_HASH_FOREACH_KEY(HashTable *ht, zend_long h, zend_string *key) 
ZEND_HASH_FOREACH_PTR(HashTable *ht, ptr)
ZEND_HASH_FOREACH_NUM_KEY(HashTable *ht, zend_long h) 
ZEND_HASH_FOREACH_STR_KEY(HashTable *ht, zend_string *key)
ZEND_HASH_FOREACH_STR_KEY_VAL(HashTable *ht, zend_string *key, zval *val)
ZEND_HASH_FOREACH_KEY_VAL(HashTable *ht, zend_long h, zend_string *key, zval *val)
```

遍历用法

```c
HashTable *ht;
zval *val;

ZEND_HASH_FOREACH_VAL(ht, val) {
    ...
} ZEND_HASH_FOREACH_END();
```


```c
uint32_t zend_hash_num_elements(HashTable *ht);  // 获取数组大小
zval* zend_hash_find(HashTable *ht, zend_string *key);  // 根据 zend_string * 作为 key 查找数组
zval* zend_hash_str_find(HashTable *ht, char *str, size_t len);  // 根据 char * 作为 key 查找数组
zval* zend_hash_index_find(HashTable *ht, zend_ulong h);  // 查找索引 h 的数组元素
void* zend_hash_find_ptr(HashTable *ht, zend_string *key);  // 同上，只是返回元素指针指向的值
void* zend_hash_str_find_ptr(HashTable *ht, char *str, size_t len);  // 跟上同类
void* zend_hash_index_find_ptr(HashTable *ht, zend_ulong h);  // 跟上同类
zend_bool zend_hash_exists(HashTable *ht, zend_string *key);  // zend_string * key 是否存在
zend_bool zend_hash_str_exists(HashTable *ht, char *str, size_t len);  // char * key 是否存在
zend_bool zend_hash_index_exists(HashTable *ht, zend_ulong h);  // 索引 h 是否存在

zend_array *HASH_OF(zval *val);  // 其实 HASH_OF 是一个宏，参数 value 可以是数组 `IS_ARRAY` 或者对象 `IS_OBJECT`，否则返回 NULL
```

# 方法和函数相关

```c
zval* zval* zend_call_method(zval *object, zend_class_entry *obj_ce, zend_function **fn_proxy, const char *function_name, size_t function_name_len, zval *retval_ptr, int param_count, zval* arg1, zval* arg2)

int zend_call_function(zend_fcall_info *fci, zend_fcall_info_cache *fci_cache)

```

# 输出

```c
void php_var_dump(zval *struc, int level);  // 类似 var_dump 方法, 需要引入 ext/standard/php_var.h
size_t php_printf(const char *format, ...);  // 格式化输出到 PHP
size_t php_output_write(const char *str, size_t len);  // 等同于 PHPWRITE 宏
spprintf(char **, max, format, var...);     使用该函数前必须先分配好第一个参数的内存空间
snprintf(char *, length, format, var...);   会动态地为第一个参数分配内存
```

# zval

## accessing a zval

```c
(zval).value.lval Z_LVAL(zval)
((zend_bool)(zval).value.lval) Z_BVAL(zval)
(zval).value.dval Z_DVAL(zval)
(zval).value.str.val Z_STRVAL(zval)
(zval).value.str.len Z_STRLEN(zval)
(zval).value.ht Z_ARRVAL(zval)
(zval).value.obj Z_OBJVAL(zval)
Z_OBJVAL(zval).handle Z_OBJ_HANDLE(zval)
Z_OBJVAL(zval).handlers Z_OBJ_HT(zval)
zend_class_entry* Z_OBJCE(zval)
HashTable* Z_OBJPROP(zval)
Z_OBJ_HT((zval))->hf Z_OBJ_HANDLER(zval, hf)
(zval).value.lval Z_RESVAL(zval)
Z_OBJDEBUG(zval,is_tmp)
(zval).type Z_TYPE(zval)

Z_*_P(zval_p) 上面的重复一遍 相当于 Z_*(*zval)
Z_*_PP(zval_pp) 上面的重复一遍 相当于 Z_*(**zval)
```

## reference count and is-ref

```c
Z_REFCOUNT(z) Retrieve reference count
Z_SET_REFCOUNT(z, rc) Set reference count to rc
Z_ADDREF(z) Increment reference count
Z_DELREF(z) Decrement reference count
Z_ISREF(z) Whether zval is a reference
Z_SET_ISREF(z) Makes zval a reference variable
Z_UNSET_ISREF(z) Resets the is-reference flag
Z_SET_ISREF_TO(z, isref) Make zval a reference is isref != 0
Z_*_P(zval_p) 1-8重复一遍 相当于 Z_*(*zval)
Z_*_PP(zval_pp) 1-8重复一遍 相当于 Z_*(**zval)
```

## setting types and values

```c
ZVAL_RESOURCE(z, l)
ZVAL_BOOL(z, b)
ZVAL_NULL(z)
ZVAL_LONG(z, l)
ZVAL_DOUBLE(z, d)
ZVAL_STR(*zval val, zeng_string* str) // 设置zval为zend_string类型
ZVAL_STRING(z, s, duplicate)
ZVAL_STRINGL(z, s, l, duplicate)
ZVAL_EMPTY_STRING(z)
ZVAL_ZVAL(z, zv, copy, dtor)
```

```c
zend_long zval_get_long(zval *val)
double zval_get_double(zval *val)
```

## allocate and initialize a zval

```c
INIT_PZVAL(zp) Set reference count and isref 0
INIT_ZVAL(z) Initialize and set NULL, no pointer
ALLOC_INIT_ZVAL(zp) Allocate and initialize a zval
MAKE_STD_ZVAL(zv) Allocate, initialize and set NULL
PZVAL_IS_REF(z)
ZVAL_COPY_VALUE(z, v)
ZVAL_COPY_VALUE(z, v)
INIT_PZVAL_COPY(z, v)
SEPARATE_ZVAL(ppzv)
SEPARATE_ZVAL_IF_NOT_REF(ppzv)
SEPARATE_ZVAL_TO_MAKE_IS_REF(ppzv)
COPY_PZVAL_TO_ZVAL(zv, pzv)
MAKE_COPY_ZVAL(ppzv, pzv)
REPLACE_ZVAL_VALUE(ppzv_dest, pzv_src, copy)
SEPARATE_ARG_IF_REF(varptr)
READY_TO_DESTROY(zv)
```
## 判断类型



# 内存分配相关

```c
viod* emalloc(size_t size)
void* ecalloc(size_t nmemb, size_t size)
void* erealloc(void *ptr, size_t size)
void* estrdup(char *str)
void* estrndup(char *str, size_t len)
void efree(void *ptr)
void *safe_emalloc(size_t nmemb, size_t size, size_t adtl)
void *STR_EMPTY_ALLOC(void)
void* pemalloc(size_t size, int persist)
void* pecalloc(size_t nmemb, size_t size, int persist)
void* perealloc(void *ptr, size_t size, int persist)
void* pestrdup(char *str, int persist)
void pefree(void *ptr, int persist)
void* safe_pemalloc(size_t nmemb, size_t size, size_t addtl, int persist)
```

# 常量相关

## flags

```c
#define CONST_CS (1<<0) /* Case Sensitive */
#define CONST_PERSISTENT (1<<1) /* Persistent */
#define CONST_CT_SUBST (1<<2) /* Allow compile-time substitution */
```

## register

```c
REGISTER_LONG_CONSTANT(name, lval, flags)
REGISTER_DOUBLE_CONSTANT(name, dval, flags)
REGISTER_STRING_CONSTANT(name, str, flags)
REGISTER_STRINGL_CONSTANT(name, str, len, flags)
REGISTER_NS_LONG_CONSTANT(ns, name, lval, flags)
REGISTER_NS_DOUBLE_CONSTANT(ns, name, dval, flags)
REGISTER_NS_STRING_CONSTANT(ns, name, str, flags)
REGISTER_NS_STRINGL_CONSTANT(ns, name, str, len, flags)
REGISTER_MAIN_LONG_CONSTANT(name, lval, flags)
REGISTER_MAIN_DOUBLE_CONSTANT(name, dval, flags)
REGISTER_MAIN_STRING_CONSTANT(name, str, flags)
REGISTER_MAIN_STRINGL_CONSTANT(name, str, len, flags)
int zend_register_constant(zend_constant *c TSRMLS_DC)
```

## get

```c
int zend_get_constant(const char *name, uint name_len, zval *result TSRMLS_DC)
zend_constant *zend_quick_get_constant(const zend_literal *key, ulong flags TSRMLS_DC)
zend_constant *zend_quick_get_constant(const zend_literal *key, ulong flags TSRMLS_DC);
```

## copy

```c
void zend_copy_constants(HashTable *target, HashTable *sourc)
void copy_zend_constant(zend_constant *c)
```

# 对象相关

## init

```c
INIT_CLASS_ENTRY(class_container, class_name, functions)
INIT_CLASS_ENTRY_INIT_METHODS(class_container, functions, handle_fcall, handle_propget, handle_propset, handle_propunset, handle_propisset)
INIT_OVERLOADED_CLASS_ENTRY(class_container, class_name, functions, handle_fcall, handle_propget, handle_propset)
INIT_NS_CLASS_ENTRY(class_container, ns, class_name, functions)
INIT_OVERLOADED_NS_CLASS_ENTRY(class_container, ns, class_name, functions, handle_fcall, handle_propget, handle_propset)
int object_init(arg)
int object_and_properties_init(arg, ce, properties)
void zend_object_std_init(zend_object *object, zend_class_entry *ce TSRMLS_DC)
zend_object_value zend_objects_new(zend_object **object, zend_class_entry *class_type TSRMLS_DC)
void zend_object_store_set_object(zval *zobject, void *object TSRMLS_DC)
void zend_object_store_ctor_failed(zval *zobject TSRMLS_DC)
zval *zend_object_create_proxy(zval *object, zval *member TSRMLS_DC)
```

## register

```c
zend_class_entry *zend_register_internal_class(zend_class_entry *class_entry TSRMLS_DC)
zend_class_entry *zend_register_internal_interface(zend_class_entry *orig_class_entry TSRMLS_DC)
int zend_register_class_alias(name, ce)
int zend_register_ns_class_alias(ns, name, ce)
```

## declaring class constants

```c
int zend_declare_class_constant(zend_class_entry *ce, const char *name, size_t name_length, zval *value TSRMLS_DC)
int zend_declare_class_constant_null(zend_class_entry *ce, const char *name, size_t name_length TSRMLS_DC)
int zend_declare_class_constant_long(zend_class_entry *ce, const char *name, size_t name_length, long value TSRMLS_DC)
int zend_declare_class_constant_bool(zend_class_entry *ce, const char *name, size_t name_length, zend_bool value TSRMLS_DC)
int zend_declare_class_constant_double(zend_class_entry *ce, const char *name, size_t name_length, double value TSRMLS_DC)
int zend_declare_class_constant_stringl(zend_class_entry *ce, const char *name, size_t name_length, const char *value, size_t value_length TSRMLS_DC)
int zend_declare_class_constant_string(zend_class_entry *ce, const char *name, size_t name_length, const char *value TSRMLS_DC)
```
## implements

```c
void zend_class_implements(zend_class_entry *class_entry TSRMLS_DC, int num_interfaces, ...)
```

## declaring property

```c
int zend_declare_property(zend_class_entry *ce, const char *name, int name_length, zval *property, int access_type TSRMLS_DC)
int zend_declare_property_ex(zend_class_entry *ce, const char *name, int name_length, zval *property, int access_type, const char *doc_comment, int doc comment_len TSRMLS_DC)
int zend_declare_property_null(zend_class_entry *ce, const char *name, int name_length, int access_type TSRMLS_DC)
int zend_declare_property_bool(zend_class_entry *ce, const char *name, int name_length, long value, int access_type TSRMLS_DC)
int zend_declare_property_long(zend_class_entry *ce, const char *name, int name_length, long value, int access_type TSRMLS_DC)
int zend_declare_property_double(zend_class_entry *ce, const char *name, int name_length, double value, int access_type TSRMLS_DC)
int zend_declare_property_string(zend_class_entry *ce, const char *name, int name_length, const char *value, int access_type TSRMLS_DC)
int zend_declare_property_stringl(zend_class_entry *ce, const char *name, int name_length, const char *value, int value_len, int access_type TSRMLS_DC)
```

## update

```c
void zend_update_class_constants(zend_class_entry *class_type TSRMLS_DC)
void zend_update_property(zend_class_entry *scope, zval *object, const char *name, int name_length, zval *value TSRMLS_DC)
void zend_update_property_null(zend_class_entry *scope, zval *object, const char *name, int name_length TSRMLS_DC)
void zend_update_property_bool(zend_class_entry *scope, zval *object, const char *name, int name_length, long value TSRMLS_DC)
void zend_update_property_long(zend_class_entry *scope, zval *object, const char *name, int name_length, long value TSRMLS_DC)
void zend_update_property_double(zend_class_entry *scope, zval *object, const char *name, int name_length, double value TSRMLS_DC)
void zend_update_property_string(zend_class_entry *scope, zval *object, const char *name, int name_length, const char *value TSRMLS_DC)
void zend_update_property_stringl(zend_class_entry *scope, zval *object, const char *name, int name_length, const char *value, int value_length TSRMLS DC)
int zend_update_static_property(zend_class_entry *scope, const char *name, int name_length, zval *value TSRMLS_DC)
int zend_update_static_property_null(zend_class_entry *scope, const char *name, int name_length TSRMLS_DC)
int zend_update_static_property_bool(zend_class_entry *scope, const char *name, int name_length, long value TSRMLS_DC)
int zend_update_static_property_long(zend_class_entry *scope, const char *name, int name_length, long value TSRMLS_DC)
int zend_update_static_property_double(zend_class_entry *scope, const char *name, int name_length, double value TSRMLS_DC)
int zend_update_static_property_string(zend_class_entry *scope, const char *name, int name_length, const char *value TSRMLS_DC)
int zend_update_static_property_stringl(zend_class_entry *scope, const char *name, int name_length, const char *value, int value_length TSRMLS_DC)
```
## read

```c
zval *zend_read_property(zend_class_entry *scope, zval *object, const char *name, int name_length, zend_bool silent TSRMLS_DC)
zval *zend_read_static_property(zend_class_entry *scope, const char *name, int name_length, zend_bool silent TSRMLS_DC)
zend_class_entry *zend_get_class_entry(const zval *zobject TSRMLS_DC)
int zend_get_object_classname(const zval *object, const char **class_name, zend_uint *class_name_len TSRMLS_DC)
char *zend_get_type_by_const(int type)
getThis()
zend_object *zend_objects_get_address(const zval *object TSRMLS_DC)
void *zend_object_store_get_object(const zval *object TSRMLS_DC)
void *zend_object_store_get_object_by_handle(zend_object_handle handle TSRMLS_DC)
```
## copy and free

```c
void zend_objects_clone_members(zend_object *new_object, zend_object_value new_obj_val, zend_object *old_object, zend_object_handle handle TSRMLS_DC)
zend_object_value zend_objects_clone_obj(zval *object TSRMLS_DC)
void zend_objects_free_object_storage(zend_object *object TSRMLS_DC)
zend_object_value zend_objects_store_clone_obj(zval *object TSRMLS_DC)
```
## destruct

```c
void zend_object_std_dtor(zend_object *object TSRMLS_DC)
void zend_objects_destroy_object(zend_object *object, zend_object_handle handle TSRMLS_DC)
void zend_objects_store_call_destructors(zend_objects_store *objects TSRMLS_DC)
void zend_objects_store_mark_destructed(zend_objects_store *objects TSRMLS_DC)
void zend_objects_store_destroy(zend_objects_store *objects)
void zend_objects_store_free_object_storage(zend_objects_store *objects TSRMLS_DC)
```
## refcount and is-ref

```c
void zend_objects_store_add_ref(zval *object TSRMLS_DC)
void zend_objects_store_del_ref(zval *object TSRMLS_DC)
void zend_objects_store_add_ref_by_handle(zend_object_handle handle TSRMLS_DC)
zend_uint zend_objects_store_get_refcount(zval *object TSRMLS_DC)
```

# GC相关

```c
#define GC_REFCOUNT(p)				(p)->gc.refcount
#define GC_TYPE(p)					(p)->gc.u.v.type
#define GC_FLAGS(p)					(p)->gc.u.v.flags
#define GC_INFO(p)					(p)->gc.u.v.gc_info
#define GC_TYPE_INFO(p)				(p)->gc.u.type_info

// * 代表 REFCOUNT/TYPE/FLAGS/INFO/TYPE_INFO
Z_GC_*(zval val)
Z_GC_*_P(*zval val)
```


