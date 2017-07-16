# Fetch API

标签（空格分隔）： 前端

---
### [Fetch api][2]的几个核心类
#### [Headers][3]
- **API**
```javascript
// Constructor
// 1. 传入js对象
new Headers({ 
    key1: value1, 
    key2: value2 
});
// 2. 传入二维数组
new Headers([
    [key1, value1],
    [key2, value2]
]);

// Methods
const content = "Hello World";
const reqHeaders = new Headers({
  "Content-Type": "text/plain",
  "Content-Length": content.length.toString(),
  "X-Custom-Header": "ProcessThisImmediately",
});

// 1. Headers.has(name) 
console.log(reqHeaders.has("Content-Type")); // true
console.log(reqHeaders.has("Set-Cookie")); // false

// 2. Headers.set(name, value) 
reqHeaders.set("Content-Type", "text/html");

// 3. Headers.append(name, value)
reqHeaders.append("X-Custom-Header", "AnotherValue");

// 4. Headers.get(name)
console.log(reqHeaders.get("Content-Length")); // 11

// 5. Headers.getAll(name)
console.log(reqHeaders.getAll("X-Custom-Header")); // ["ProcessThisImmediately", "AnotherValue"]

// 6. Headers.delete(name)
reqHeaders.delete("X-Custom-Header");
console.log(reqHeaders.getAll("X-Custom-Header")); // []

// 7. Headers.keys()
for(let k of reqHeaders.keys()) {
    console.log(k);   
    // 输出结果：
    // content-length  
    // content-type
}

// 8. Headers.values()
for (let v of reqHeaders.values()) {
    console.log(v);   
    // 输出结果：
    // 11    
    // text/html
}

// 9. Headers.entries()
for(let en of reqHeaders.entries()) {
    console.log(`${en[0]}: ${en[1]}`);
    // 输出结果：
    // content-length: 11
    // content-type: text/html
}
```
- Headers对象有一个特殊的``guard``属性，这个属性没有暴露给Web，但是它影响到哪些内容可以在Headers对象中被改变。
|guard的值|Headers中的行为|备注|
|--|--|--|
|none||默认值，Headers(init)构造器别调用是，guard被赋予此值|
|request|调用Headers.append()、Headers.delete()或Headers.set()时，``被禁止的首部``会被无视；|Request.headers的guard属性值|
|request-no-cors|调用Headers.append()、Headers.delete()或Headers.set()时，非``CORS安全的请求首部字段``会被无视；|mode值等于``no-cors``的Request中headers的guard属性值|
|response|调用Headers.append()、Headers.delete()或Headers.set()时，非``CORS安全的响应首部字段``会被无视；|Response.headers的guard属性值|
|immutable|headers是只读的；当调用Headers.append()、Headers.delete()或Headers.set()时，会抛出TypeError；|常用于ServiceWorkers|


- 被禁止的首部字段：
    - Accept-Charset
    - Accept-Encoding
    - Access-Control-Request-Headers
    - Access-Control-Request-Method
    - Connection
    - Content-Length
    - Cookie
    - Cookie2
    - Date
    - DNT
    - Expect
    - Host
    - Keep-Alive
    - Origin
    - Referer
    - TE
    - Trailer
    - Transfer-Encoding
    - Upgrade
    - Via

- 对CORS安全的请求首部字段集合：
    - Accept
    - Accept-Language
    - Content-Language
    - Content-Type：且值必须是 `application/x-www-form-urlencoded`, `multipart/form-data`, 或 `text/plain`之一。

- 对CORS安全的响应首部字段集合：
    - Cache-Control
    - Content-Language
    - Content-Type
    - Expires
    - Last-Modified
    - Pragma
#### [Request][4]
- API
```javascript  
// Constructor
new Request(input, init);

// input的可能的值：
//  1. URL实例 
new Request(new URL('localhost:8080'));

//  2. Request实例，这种用法通常见于ServiceWorkers
new Request(new Request('/test'));

//  3. String(url) 
new Request('/test');

// init是一个js对象，提供request初始化的参数
new Request("/uploadImage", {
  method: "POST",
  headers: {
    "Content-Type": "image/png",
  },
  body: "<image data>"
});
```
- init中的其他参数：
    - mode: ``same-origin`` | ``cors`` | ``no-cors（默认值）``
        - same-origin： 只能发起同源的请求
        - cors：允许跨域的请求。
        - no-cors：允许来自CDN的脚本、其他域的图片和其他一些跨域资源，但请求的method是``HEAD`` | ``GET`` | ``POST``之一。此外，任何ServiceWorkers拦截了这些请求，它不能随意添加或改写任何非CORS安全的请求首部字段。而且Response中的任何属性不能被访问，这保证了ServiceWorkers不会因为任何跨域下的安全问题而导致隐私信息泄露。?
    - credentials（与XHR的``withCredentials``标志相同）: ``omit（默认值）`` | ``include`` | ``same-origin``
        - omit：不发送cookie。
        - include：所有请求都会发送cookie，包括跨域请求。
        - same-origin：只有同源的请求才发送cookie。
#### [Response][5]
- type: ``basic`` | ``cors`` | ``error`` | ``opaque``
    - basic：正常的同源请求，包含``Set-Cookie``和``Set-Cookie2``以外的所有headers。
    - cors：Response从一个合法的跨域请求获得，一部分header和body可读。
    - error：网络错误，当Response是从Response.error()中得到时，就是这种类型。此时status是0，headers是空且不可写。且会导致fetch()函数的Promise被reject并回调出一个TypeError。
    - opaque：通过``no-cors的Request``请求的跨域资源的Response。
#### [Body][6]
    无论Request还是Response都可能带着body。Request和Response（通过fetch()方法）都能够自动识别自己的content-type（body的数据类型）。如果开发者没有设置headers中的Content-Type，Request也可以自动设置。

- body可能是以下数据类型之一：
    - ArrayBuffer
    - ArrayBufferView (Uint8Array and friends)
    - Blob/File
    - String
    - URLSearchParams
    - FormData
- Body中提供了以下方法，来处理对应的数据类型，这些方法都返回一个Promise对象。由于Request和Response都继承了Body接口，所以他们都包含以下方法。
    - arrayBuffer()
    - blob()
    - json()
    - text()
    - formData()

- 不管是Request还是Response中的body只能被读取一次。它们中有一个属性``bodyUsed``标识body是否被读取。


#### 参考文献
1. https://hacks.mozilla.org/2015/03/this-api-is-so-fetching/
2. https://fetch.spec.whatwg.org/
3. https://github.com/github/fetch/blob/master/fetch.js

  [1]: https://github.com/github/fetch/blob/master/fetch.js
  [2]: https://fetch.spec.whatwg.org/#fetch-api
  [3]: https://fetch.spec.whatwg.org/#headers-class
  [4]: https://fetch.spec.whatwg.org/#request-class
  [5]: https://fetch.spec.whatwg.org/#response-class
  [6]: https://fetch.spec.whatwg.org/#body-mixin
  [7]: https://hacks.mozilla.org/2015/03/this-api-is-so-fetching/