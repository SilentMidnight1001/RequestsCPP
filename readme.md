# Requests 库使用文档

## 简介

Requests 是一个基于 libcurl 的 HTTP 请求库，提供简洁易用的 API 接口，支持 GET 和 POST 请求、JSON 数据交互、自定义请求头、Cookie 管理等功能。

## 依赖

- libcurl
- winKeyPressH.hpp（提供字典容器和 JSON 解析功能）

## 快速开始

```cpp
#include "requests.hpp"

int main()
{
    // GET 请求
    requests::get("https://api.example.com/users");
    
    // POST 请求
    requests::post("https://api.example.com/users");
    
    return 0;
}
```

---

## RequestOptions 结构体

请求配置选项：

| 成员 | 类型 | 说明 |
|------|------|------|
| `params` | `wkp::Dict<string, string>` | URL 查询参数 |
| `headers` | `wkp::Dict<string, string>` | 自定义请求头 |
| `cookies` | `wkp::Dict<string, string>` | Cookie |
| `data` | `wkp::Dict<string, string>` | application/x-www-form-urlencoded 表单数据 |
| `json` | `std::string` | 原始 JSON 请求体 |
| `connectTimeoutMs` | `long` | 连接超时时间（毫秒），默认 5000 |
| `timeoutMs` | `long` | 请求超时时间（毫秒），默认 10000 |
| `allowRedirects` | `bool` | 是否允许重定向，默认 true |
| `verify` | `bool` | 是否验证 SSL 证书，默认 true |

---

## Requests 类（父类）

### 成员变量

| 变量 | 类型 | 说明 |
|------|------|------|
| `res` | `CURLcode` | libcurl 执行结果码 |
| `response` | `std::string` | 响应体文本 |
| `http_status` | `long` | HTTP 状态码 |

### 方法

#### GET 请求

```cpp
void sendGetRequest(
    const std::string &url,
    const RequestOptions &opt = {},
    const std::string &certificatePath = "./ssh/cacert.pem"
);
```

**示例：**
```cpp
requests::Requests req;
RequestOptions opt;
opt.params["id"] = "123";
opt.params["type"] = "user";
opt.headers["Authorization"] = "Bearer token123";

req.sendGetRequest("https://api.example.com/get", opt);
if (req.res == CURLE_OK) {
    std::cout << "状态码: " << req.status_code() << std::endl;
    std::cout << "响应: " << req.text() << std::endl;
}
```

#### POST 请求

```cpp
void sendPostRequest(
    const std::string &url,
    const RequestOptions &opt = {},
    const std::string &certificatePath = "./ssh/cacert.pem"
);
```

**示例（表单数据）：**
```cpp
RequestOptions opt;
opt.data["username"] = "john";
opt.data["password"] = "123456";

req.sendPostRequest("https://api.example.com/login", opt);
```

**示例（JSON 数据）：**
```cpp
RequestOptions opt;
opt.json = R"({"name":"John","age":30})";

req.sendPostRequest("https://api.example.com/users", opt);
```

#### 响应处理方法

| 方法 | 返回类型 | 说明 |
|------|----------|------|
| `text()` | `std::string` | 获取原始响应文本 |
| `content()` | `std::vector<unsigned char>` | 获取原始二进制响应 |
| `json()` | `wkp::Dict<string, wkp::JsonValue>` | 获取 JSON 格式响应（自动解析） |
| `isJson()` | `bool` | 检查响应是否为有效 JSON |
| `status_code()` | `int` | 获取 HTTP 状态码 |

---

## 便捷对象

### RequestsGet

```cpp
requests::RequestsGet get;
```

支持链式调用：

```cpp
// 基本用法
requests::get("https://api.example.com/users");

// 带选项
RequestOptions opt;
opt.params["page"] = "1";
requests::get("https://api.example.com/users", opt);

// 链式调用（返回自身引用）
auto& result = requests::get("https://api.example.com/users");
std::cout << result.text() << std::endl;
```

### RequestsPost

```cpp
requests::RequestsPost post;
```

```cpp
// 基本用法
requests::post("https://api.example.com/users");

// 带选项
RequestOptions opt;
opt.json = R"({"name":"John"})";
requests::post("https://api.example.com/users", opt);
```

---

## 完整示例

```cpp
#include "requests.hpp"
#include <iostream>

int main()
{
    // 示例 1: 简单 GET 请求
    requests::get("https://httpbin.org/get");
    std::cout << "状态码: " << requests::get.status_code() << std::endl;
    std::cout << "响应: " << requests::get.text() << std::endl;

    // 示例 2: GET 请求带参数和请求头
    RequestOptions opt1;
    opt1.params["q"] = "cpp";
    opt1.params["page"] = "1";
    opt1.headers["User-Agent"] = "MyApp/1.0";
    opt1.timeoutMs = 30000;

    requests::get("https://httpbin.org/get", opt1);
    
    if (requests::get.isJson()) {
        auto json = requests::get.json();
        // 处理 JSON 数据...
    }

    // 示例 3: POST 表单数据
    RequestOptions opt2;
    opt2.data["username"] = "john_doe";
    opt2.data["password"] = "secret123";

    requests::post("https://httpbin.org/post", opt2);
    std::cout << "表单响应: " << requests::post.text() << std::endl;

    // 示例 4: POST JSON 数据
    RequestOptions opt3;
    opt3.json = R"({
        "title": "Hello World",
        "content": "This is a test post",
        "author": "John"
    })";
    opt3.headers["X-API-Key"] = "your-api-key";

    requests::post("https://api.example.com/posts", opt3);

    // 示例 5: 带 Cookie 的请求
    RequestOptions opt4;
    opt4.cookies["session_id"] = "abc123def456";
    opt4.cookies["user_pref"] = "dark_mode";

    requests::get("https://example.com/dashboard", opt4);

    // 示例 6: 不使用 SSL 证书验证（仅测试环境）
    RequestOptions opt5;
    opt5.verify = false;  // 跳过证书验证

    requests::get("https://self-signed.badssl.com", opt5);

    // 示例 7: 使用自定义 CA 证书
    requests::get(
        "https://api.example.com/secure",
        RequestOptions{},           // 默认选项
        "./my-custom-cacert.pem"    // 自定义证书路径
    );

    return 0;
}
```

---

## 注意事项

1. **全局初始化**：libcurl 全局环境自动初始化，无需手动调用
2. **证书文件**：默认证书路径为 `./ssh/cacert.pem`，可通过参数自定义
3. **线程安全**：每个 `Requests` 实例独立，但全局初始化是线程安全的
4. **JSON 解析**：依赖 winKeyPressH 的 JSON 解析功能，无效 JSON 会返回空字典
5. **响应存储**：响应内容存储在 `response` 成员变量中，可通过 `text()` 获取
6. **错误处理**：检查 `res` 成员变量判断请求是否成功（`res == CURLE_OK`）

---

## 错误码参考

| 错误码 | 含义 |
|--------|------|
| `CURLE_OK` | 请求成功 |
| `CURLE_COULDNT_CONNECT` | 无法连接服务器 |
| `CURLE_OPERATION_TIMEDOUT` | 操作超时 |
| `CURLE_SSL_CACERT` | SSL 证书问题 |