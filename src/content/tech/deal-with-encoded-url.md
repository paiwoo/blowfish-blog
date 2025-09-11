---
title: "拿到编码的 URL 怎么办"
weight: 6
draft: false
# categories: ["tech"]
tags: ["c#"]
# series: ["tech"]
# series_order: 4
---


---

🔗 拿到编码的 URL 怎么办？3 秒搞定！

> 关键词：URL 编码、Percent-Encoding、C#、HttpUtility、WebUtility  

---

## 1. 现象 🤔  
从日志/接口里拷出来的链接长这样：  
```
https%3A%2F%2Ftestxiafa.ccppayment.top%2Fcasho%3ForderId%3Ddeee7020226c67d9d14a7c03ab8e8efa
```  
直接扔浏览器？404。  
直接 `HttpClient.GetAsync(url)`？DNS 挂掉。  

---

## 2. 原因 🔍  
`:` → `%3A`  
`/` → `%2F`  
`?` → `%3F`  
**整串 URL 被 Percent-Encoding 了！**  

---

## 3. 解法 ✅  
**一句话：先解码，再使用。**  

### .NET 6+ / .NET Core（跨平台）
```csharp
using System.Net;

string raw = "https%3A%2F%2Ftestxiafa.ccppayment.top%2Fcasho%3ForderId%3D...";
string url  = WebUtility.UrlDecode(raw);   // 解码
Console.WriteLine(url);                    // https://testxiafa.ccppayment.top/casho?orderId=...
```

### 经典 .NET Framework
```csharp
using System.Web;

string url = HttpUtility.UrlDecode(raw);
```

---

## 4. 常见坑 ⚠️  
| 操作 | 不解码的后果 |
|---|---|
| `new Uri(raw)` | 主机名变成 `https%3a` → 无法解析 |
| `HttpClient.GetAsync(raw)` | 请求整串百分号地址 → DNS 失败 |
| `Process.Start(raw)` | 浏览器报“地址无效” |

---

## 5. 一句话记忆 🧠  
**“% 看起来不舒服，先 UrlDecode 再上路！”**

---

## 6. 复制即可用 📦  
```csharp
string FixEncodedUrl(string encoded) =>
    string.IsNullOrWhiteSpace(encoded) 
        ? encoded 
        : WebUtility.UrlDecode(encoded);
```

---

## 7. 相关链接 🔗  
- [RFC 3986 - Percent-Encoding](https://tools.ietf.org/html/rfc3986)  
- [Microsoft Docs - WebUtility.UrlDecode](https://learn.microsoft.com/dotnet/api/system.net.webutility.urldecode)

---

**END**