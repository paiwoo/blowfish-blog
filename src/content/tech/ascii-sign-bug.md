---
title: "验签失败？C#默认排序坑"
weight: 6
draft: false
# categories: ["tech"]
tags: ["c#"]
# series: ["tech"]
# series_order: 4
---

🔐 验签失败？SortedDictionary 默认排序 ≠ ASCII 序 —— .NET 4 踩坑记录

> 环境：.NET Framework 4.x  
> 关键词：SortedDictionary、StringComparer.Ordinal、验签、ASCII 字节序、签名串

---

## 1. 问题现象 🚨

对接第三方支付网关时，本地生成的签名串与网关验签用的串**字节不一致**，导致**验签失败**。

```csharp
// 看起来一切正常，却永远验签失败
var dict = new SortedDictionary<string, string>
{
    { "productId", "P001" },
    { "transId",   "T123" },
    { "merNo",     "M999" },
    { "orderNo",   "O456" },
    { "transAmt",  "1.00" },
    { "orderDate", "20250910" },
    { "respCode",  "00" },
    { "respDesc",  "success" }
};
string signStr = string.Join("&",
    dict.Where(kv => !string.IsNullOrEmpty(kv.Value))
        .Select(kv => $"{kv.Key}={kv.Value}"));
```

---

## 2. 排查过程 🔍

| 步骤 | 操作 | 结果 |
|---|---|---|
| ① | 打印 `signStr` | `merNo=M999&orderDate=20250910&orderNo=O456&productId=P001&respCode=00&respDesc=success&transAmt=1.00&transId=T123` |
| ② | 与网关日志比对 | 网关期望 `orderDate` 在 `orderNo` **之后** |
| ③ | 16 进制对比 | 发现 **大写字母排在小写字母前**（`P` < `o`）—— 典型的 **ASCII 码序** |
| ④ | 反编译 `SortedDictionary` | 默认比较器为 `Comparer<TKey>.Default`，对 `string` 即 `StringComparer.Ordinal`，**但**服务器若切换区域（如 tr-TR）会表现不同；**更重要的是**，`SortedDictionary` 的“有序”语义≠**显式 Ordinal 排序** |
| ⑤ | 结论 | **默认排序并非可靠的 ASCII 字节序** |

---

## 3. 根因总结 ⚠️

1. `SortedDictionary<TKey,TValue>` 默认按 `Comparer<TKey>.Default` 排序，对 `string` 即 `StringComparer.Ordinal`，**看似**符合 ASCII，但：
   - 一旦宿主线程区域文化被修改，**行为可能变化**；
   - **可读性差**：后续维护者难以一眼看出“这里是按 ASCII 排序”。
2. 支付网关普遍按 **原始 ASCII 码值**排序，即 `StringComparer.Ordinal`。
3. 两端排序不一致 → **签名串字节级差异** → 验签失败。

---

## 4. 最小复现代码 🧪

```csharp
var list = new[] { "orderDate", "orderNo" };
Array.Sort(list);                       // 默认文化序
Console.WriteLine(string.Join(", ", list));
// 输出：orderDate, orderNo   （本地 en-US）

Array.Sort(list, StringComparer.Ordinal);
Console.WriteLine(string.Join(", ", list));
// 输出：orderNo, orderDate   （严格 ASCII 序）
```

---

## 5. 正确姿势 ✅

**显式指定 `StringComparer.Ordinal`**，并封装成**一次性扩展方法**，杜绝手滑。

```csharp
public static class SignExtensions
{
    /// <summary>
    /// 按 ASCII 字节序(Ordinal)拼成 k=v&k=v，跳过空值。
    /// </summary>
    public static string ToQueryString(this IDictionary<string, string> src,
                                    bool skipEmpty = true)
    {
        if (src == null) throw new ArgumentNullException(nameof(src));

        return string.Join("&",
                    src.Where(kv => !skipEmpty || !string.IsNullOrEmpty(kv.Value))
                       .OrderBy(kv => kv.Key, StringComparer.Ordinal)
                       .Select(kv => string.Concat(kv.Key, "=", kv.Value)));
    }
}
```

调用处一行代码，**永远不再踩排序坑**：

```csharp
var dict = new Dictionary<string, string>
{
    ["productId"] = productId,
    ["transId"]   = transId,
    ["merNo"]     = merNo,
    ["orderNo"]   = orderNo,
    ["transAmt"]  = transAmt,
    ["orderDate"] = orderDate,
    ["respCode"]  = respCode,
    ["respDesc"]  = respDesc
};

string signatureStr = dict.ToQueryString();   // 严格 ASCII 字节序
```

---

## 6. 一键检查表 📋

| 检查项 | 是否通过 |
|---|---|
| 使用 `OrderBy(..., StringComparer.Ordinal)` | ✅ |
| 排除空值参数 | ✅ |
| Key 大小写已与网关约定一致 | ✅ |
| 最终串与网关示例 **字节一致**（可转 16 进制比对） | ✅ |
| 封装为扩展方法 + 单元测试（Golden Master） | ✅ |

---

## 7. 结论 🎯

- **SortedDictionary 默认排序 ≠ 可靠的 ASCII 序**  
- **任何涉及“字节级一致”的签名、验签场景，务必显式使用 `StringComparer.Ordinal`**  
- **把“过滤 → 排序 → 拼接”封装成一次性扩展方法，并配套单元测试，才是最佳实践**