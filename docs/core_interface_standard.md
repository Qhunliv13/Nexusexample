# NXLD 核心接口标准

## 概述

本文档说明 NXLD 引擎核心接口标准，包括配置文件解析、插件加载与管理等核心功能接口。

## 1. 配置文件解析接口 (`nxld_parser.h`)

### 1.1 配置结构体

```c
typedef struct {
    int lock_mode;                          // 锁模式：0=关闭，1=开启
    int max_root_plugins;                   // 最大根插件数
    char** enabled_root_plugins;            // 启用的根插件路径列表
    size_t enabled_root_plugins_count;      // 插件路径数量
    char** virtual_parent_keys;             // 子插件路径列表
    char** virtual_parent_values;           // 父插件路径列表
    size_t virtual_parent_count;            // 虚拟父级映射数量
} nxld_config_t;
```

### 1.2 解析函数

#### `nxld_parse_file`
解析 NXLD 配置文件（.nxld 格式，支持 UTF-8 编码）

**函数签名：**
```c
nxld_parse_result_t nxld_parse_file(const char* file_path, nxld_config_t* config);
```

**参数：**
- `file_path`: 配置文件路径
- `config`: 输出配置结构体指针

**返回值：**
- `NXLD_PARSE_SUCCESS`: 解析成功
- `NXLD_PARSE_FILE_ERROR`: 文件读取错误
- `NXLD_PARSE_ENCODING_ERROR`: 编码错误
- `NXLD_PARSE_MISSING_SECTION`: 缺少必填配置段
- `NXLD_PARSE_INVALID_LOCK_MODE`: LockMode 值非法
- `NXLD_PARSE_INVALID_MAX_PLUGINS`: MaxRootPlugins 值非法
- `NXLD_PARSE_EMPTY_PLUGINS`: EnabledRootPlugins 为空
- `NXLD_PARSE_PLUGIN_NOT_FOUND`: 插件文件未找到
- `NXLD_PARSE_PLUGIN_INVALID_FORMAT`: 插件文件格式不符合运行系统要求
- `NXLD_PARSE_VIRTUAL_PARENT_INVALID`: 虚拟父级配置中的插件路径不在 EnabledRootPlugins 中
- `NXLD_PARSE_MEMORY_ERROR`: 内存分配错误

#### `nxld_config_free`
释放配置结构体内存

**函数签名：**
```c
void nxld_config_free(nxld_config_t* config);
```

#### `nxld_get_error_message`
获取解析结果错误信息

**函数签名：**
```c
const char* nxld_get_error_message(nxld_parse_result_t result);
```

## 2. 插件加载与管理接口 (`nxld_plugin.h`)

### 2.1 数据结构

#### 参数类型枚举
```c
typedef enum {
    NXLD_PARAM_TYPE_VOID = 0,      // void 类型
    NXLD_PARAM_TYPE_INT,            // int 类型
    NXLD_PARAM_TYPE_LONG,           // long 类型
    NXLD_PARAM_TYPE_FLOAT,          // float 类型
    NXLD_PARAM_TYPE_DOUBLE,          // double 类型
    NXLD_PARAM_TYPE_CHAR,            // char 类型
    NXLD_PARAM_TYPE_POINTER,         // 指针类型
    NXLD_PARAM_TYPE_STRING,          // 字符串类型
    NXLD_PARAM_TYPE_VARIADIC,        // 可变参数类型
    NXLD_PARAM_TYPE_ANY,             // 任意类型
    NXLD_PARAM_TYPE_UNKNOWN          // 未知类型
} nxld_param_type_t;
```

#### 参数数量类型枚举
```c
typedef enum {
    NXLD_PARAM_COUNT_FIXED = 0,     // 固定数量
    NXLD_PARAM_COUNT_VARIABLE,       // 可变数量
    NXLD_PARAM_COUNT_UNKNOWN         // 未知数量
} nxld_param_count_type_t;
```

#### 插件元数据结构体
```c
typedef struct {
    char uid[65];                    // 唯一标识符（64位随机字符串）
    char* plugin_name;               // 插件名称
    char* plugin_version;             // 插件版本
    char* plugin_path;                // 插件文件路径
    nxld_interface_info_t* interfaces; // 接口信息数组
    size_t interface_count;          // 接口数量
    void* handle;                     // 动态库句柄
} nxld_plugin_t;
```

### 2.2 核心函数

#### `nxld_plugin_load`
加载插件

**函数签名：**
```c
nxld_plugin_load_result_t nxld_plugin_load(const char* plugin_path, nxld_plugin_t* plugin);
```

**参数：**
- `plugin_path`: 插件文件路径
- `plugin`: 输出插件结构体指针

**返回值：**
- `NXLD_PLUGIN_LOAD_SUCCESS`: 加载成功
- `NXLD_PLUGIN_LOAD_FILE_ERROR`: 文件错误
- `NXLD_PLUGIN_LOAD_SYMBOL_ERROR`: 符号未找到
- `NXLD_PLUGIN_LOAD_METADATA_ERROR`: 元数据错误
- `NXLD_PLUGIN_LOAD_MEMORY_ERROR`: 内存分配错误

#### `nxld_plugin_unload`
卸载插件

**函数签名：**
```c
void nxld_plugin_unload(nxld_plugin_t* plugin);
```

#### `nxld_plugin_free`
释放插件结构体内存

**函数签名：**
```c
void nxld_plugin_free(nxld_plugin_t* plugin);
```

#### `nxld_plugin_get_error_message`
获取插件加载结果错误信息

**函数签名：**
```c
const char* nxld_plugin_get_error_message(nxld_plugin_load_result_t result);
```

#### `nxld_plugin_generate_metadata_file`
生成插件元数据文件（.nxp 格式）

**函数签名：**
```c
int nxld_plugin_generate_metadata_file(const nxld_plugin_t* plugin, const char* output_path);
```

**返回值：**
- `0`: 成功
- 非 `0`: 失败

## 3. 插件批量加载器接口 (`nxld_plugin_loader.h`)

### 3.1 核心函数

#### `nxld_load_plugins_from_config`
按配置顺序批量加载所有插件

**函数签名：**
```c
int nxld_load_plugins_from_config(const nxld_config_t* config, 
                                   const char* config_file_path, 
                                   nxld_plugin_t** plugins, 
                                   size_t* loaded_count);
```

**参数：**
- `config`: 配置结构体指针
- `config_file_path`: 配置文件路径
- `plugins`: 输出插件数组指针
- `loaded_count`: 输出成功加载的插件数量

**返回值：**
- `0`: 成功
- `-1`: 失败

#### `nxld_free_plugins`
释放插件数组内存

**函数签名：**
```c
void nxld_free_plugins(nxld_plugin_t* plugins, size_t count);
```

## 4. 注意事项

1. **内存管理**
   - 所有通过接口分配的内存必须使用对应的 `free` 函数释放
   - 配置结构体和插件结构体使用完毕后必须调用相应的释放函数

2. **错误处理**
   - 所有函数调用后应检查返回值
   - 使用 `nxld_get_error_message` 和 `nxld_plugin_get_error_message` 获取详细错误信息

