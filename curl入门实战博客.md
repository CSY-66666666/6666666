# curl 入门实战：用四次命令读懂 HTTP 请求

> 课程作业记录：在 Windows PowerShell 中使用 curl 完成四次网络请求实验，截图留存。

## 一、什么是 curl

**curl（Client URL）** 是一款功能强大的命令行工具，用于在客户端与服务器之间传输数据。它支持 HTTP、HTTPS、FTP、SFTP 等 20 多种协议，是开发者进行 **API 调试、网络问题排查和自动化脚本编写** 的核心工具。

简单理解：浏览器是"带界面的网络请求工具"，而 curl 就是"命令行版的网络请求工具"——我们敲一行命令，它就替我们把请求发给服务器，并把服务器的响应原样打印出来。

---

## 二、实验一：GET 请求，直接拿到网页源码

**命令：**

```powershell
curl.exe https://example.com
```

**截图位置：**【插入截图：第一张，直接返回 HTML 源码的那一张】

**结果：** 终端直接打印出 example.com 的完整 HTML 源码，包括 `<!doctype html>`、页面标题 `Example Domain` 和页面正文。

**逐行理解：**

- 不加任何参数时，curl 默认发送 **GET 请求**；
- GET 请求的作用就是"向服务器要一份资源"，这里要的就是这个网页的 HTML；
- 服务器把网页内容（响应体 body）返回，curl 原样输出到终端。

---

## 三、实验二：只看响应头（HEAD 请求）

**命令：**

```powershell
curl.exe -I https://example.com
```

**截图位置：**【插入截图：第二张，只返回响应头的那一张】

**参数解释：**

- `-I`（大写 i）：发送 **HEAD 请求**。HEAD 和 GET 的区别是：GET 把网页内容全部下载回来，HEAD **只索要响应头，不下载网页正文**，用来快速查看服务器信息。

**响应头逐行解读：**

| 响应头 | 含义 |
| --- | --- |
| `HTTP/1.1 200 OK` | 请求成功，状态码 200 |
| `Date: Sat, 31 Oct 2026 05:41:27 GMT` | 服务器生成响应的时间（GMT 时区） |
| `Content-Type: text/html; charset=utf-8` | 返回内容是 HTML 网页，编码为 UTF-8 |
| `Connection: keep-alive` | 保持 TCP 连接，方便后续复用 |
| `Server: cloudflare` | 服务器使用了 Cloudflare 网络服务 |
| `last-modified: Thu, 29 Oct 2026 13:15:03 GMT` | 网页最后修改时间 |
| `allow: GET, HEAD` | 该资源只允许 GET 和 HEAD 方法 |
| `Accept-Ranges: bytes` | 支持按字节分段下载 |
| `cf-cache-status: HIT` | 请求命中了 Cloudflare 缓存 |

---

## 四、实验三：POST 请求，提交 JSON 数据

**命令：**

```powershell
curl.exe -X POST -H "Content-Type: application/json" -d '{"name":"test","role":"student"}' https://httpbin.org/post
```

**截图位置：**【插入截图：第三张，POST 到 httpbin.org 的那一张】

**参数解释：**

- `-X POST`：指定请求方法为 POST（POST 用于向服务器提交数据）；
- `-H "Content-Type: application/json"`：添加一个请求头，告诉服务器"我提交的内容是 JSON 格式"；
- `-d '{...}'`：携带请求体（要提交的数据）；
- `https://httpbin.org/post`：这是一个专门用来测试 HTTP 请求的公共服务，它会把我们发出的请求**原样回显**回来，方便检查请求到底发成了什么样。

**回显结果解读：**

- `"method": "POST"`：服务器确认收到的是 POST 请求；
- `"headers"`：回显了我们发送的请求头，包括 `Content-Type: application/json` 和 `User-Agent: curl/8.21.0`（告诉服务器客户端是 curl 8.21.0）；
- `"origin": "202.193.79.23"`：我们这台机器出口的公网 IP；
- `"url"`：回显请求地址。

### 踩坑记录：`"json": null` 是怎么回事？

仔细看回显结果，`"json": null`，而 `data` 里的内容变成了 `{name:test,role:student}`——双引号全部丢失了。这是 **PowerShell 传递参数时的引号转义问题**：单引号里的双引号在交给 curl.exe 的过程中被剥离，导致服务器收到的不再是合法 JSON，所以 httpbin 无法解析出 `json` 字段。

**修正写法（推荐用变量存放 JSON，避免引号嵌套）：**

```powershell
$body = '{"name":"test","role":"student"}'
curl.exe -X POST -H "Content-Type: application/json" --data-raw $body https://httpbin.org/post
```

或者在双引号字符串里用 `\"` 转义内部双引号：

```powershell
curl.exe -X POST -H "Content-Type: application/json" -d "{\"name\":\"test\",\"role\":\"student\"}" https://httpbin.org/post
```

修正后再次请求，回显中的 `"json"` 字段就会正确显示为解析后的对象。

---

## 五、实验四：只输出 HTTP 状态码

**命令：**

```powershell
curl.exe -s -o NUL -w "%{http_code}\n" https://example.com
```

**截图位置：**【插入截图：第四张，只输出 `200` 的那一张】

**参数解释：**

- `-s`：静默模式（silent），不显示下载进度条和多余提示；
- `-o NUL`：把返回的网页正文丢弃（Windows 下 `NUL` 是空设备，相当于 Linux 的 `/dev/null`），不保存、不打印；
- `-w "%{http_code}\n"`：请求结束后按自定义格式输出信息，`%{http_code}` 是 curl 内置变量，代表 **HTTP 状态码**，`\n` 表示输出后换行。

**结果：** 终端只打印出一行 `200`，表示网站正常响应。这种写法在自动化脚本里非常常用——脚本只需要判断状态码是不是 `200`，就能知道网站是否可达，不用关心网页内容。

---

## 六、常用参数速查表

| 参数 | 作用 |
| --- | --- |
| （无参数） | 默认发送 GET 请求，打印响应体 |
| `-I` | 发送 HEAD 请求，只获取响应头 |
| `-X POST` | 指定请求方法（GET / POST / PUT / DELETE 等） |
| `-H "头: 值"` | 添加自定义请求头 |
| `-d "数据"` | 携带请求体，提交表单或 JSON |
| `--data-raw 变量` | 原样提交数据（适合在 PowerShell 里传 JSON） |
| `-s` | 静默模式，关闭进度输出 |
| `-o 文件名` | 把响应体保存到文件（`NUL` 表示丢弃） |
| `-w "格式串"` | 请求结束后输出自定义信息，如 `%{http_code}` |

## 七、小结

通过这四次实验可以看到：curl 的本质就是"手动构造一个 HTTP 请求发给服务器，再把响应拿回来"。GET 用来取数据，POST 用来交数据，`-I` 只看头，`-w` 只取状态码。掌握这些基础参数，后续调试 API、排查网络问题时都可以直接用它。
