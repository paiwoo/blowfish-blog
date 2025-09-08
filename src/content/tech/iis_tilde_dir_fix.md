---
title: "IIS “波浪号目录枚举”漏洞解决"
weight: 5
draft: false
# categories: ["tech"]
tags: ["IIS"]
# series: ["tech"]
# series_order: 4
---


**主题**：解决微软 IIS “波浪号目录枚举”漏洞（IIS 短文件名泄露）  
**更新**：2025-09-08  

---

## 🛠️ 漏洞概述  
| 项目 | 说明 |  
|---|---|  
| **名称** | IIS 短文件名泄露 / 波浪号目录枚举 |  
| **CVE** | 无单独编号，微软官方归类为“信息泄露” |  
| **影响** | 攻击者利用 `~1` 风格短名可暴力枚举服务器文件，进而猜解后台、备份或敏感文件 |  
| **适用** | IIS 5.x–10.x + NTFS 8.3 命名机制（默认开启） |  

---

## 📝 原理简述  
1. **NTFS 8.3 机制**：为兼容 16-bit DOS，自动为长文件名生成“8.3”短文件名，如 `LoginPage.aspx` → `LOGINP~1.ASP`。  
2. **IIS 响应差异**：对 `*~1*` 请求返回不同状态码：  
   - **404** → 短名存在  
   - **400** → 短名不存在  
3. **攻击利用**：利用 404/400 差异可逐位爆破完整目录结构。  

---

## 🛠️ 修复策略  
| 顺序 | 动作 | 目的 | 是否必须重启 |  
|---|---|---|---|  
| ① | 关闭 NTFS 8.3 生成 | 根治新文件不再产生短名 | **是** |  
| ② | 重建 Web 目录 | 去除已存在短名 | 否（卷内移动即可） |  
| ③ | URL Rewrite 拦截 | 在线防护，阻断爆破请求 | 否（`iisreset` 即可） |  

---

## 🛠️ 详细步骤  

### 1️⃣ 关闭 NTFS 8.3 支持（核心步骤）  
```cmd
:: 查询当前状态
fsutil 8dot3name query

:: 全局关闭（1=禁用，0=启用）
fsutil 8dot3name set 1
```
**注册表等价**：  
`HKLM\SYSTEM\CurrentControlSet\Control\FileSystem\NtfsDisable8dot3NameCreation = 1`  

> ⚠️ 修改后 **必须重启操作系统** 才能对新生成文件生效 。  

---

### 2️⃣ 清除已有短文件名 —— 目录重建法  
1. **完整复制站点到临时目录**（带权限）  
   ```cmd
   robocopy D:\www D:\www_tmp /COPYALL /E /MOVE
   ```
2. **删除原目录**  
   ```cmd
   rd D:\www
   ```
3. **把临时目录重命名回原路径**  
   ```cmd
   ren D:\www_tmp www
   ```
> 同一卷内移动不会重新生成 8.3 名，旧短名即被甩掉 。  

---

### 3️⃣ URL Rewrite 在线拦截（不重启系统，即时生效）  
1. **安装 IIS URL Rewrite 模块**（若未装）：  
   下载：https://www.iis.net/downloads/microsoft/url-rewrite  
2. **在站点 `web.config` 的 `<system.webServer>` 节点内追加**：  
   ```xml
   <rewrite>
     <rules>
       <rule name="BlockShortName" stopProcessing="true">
         <match url=".*" />
         <conditions logicalGrouping="MatchAny">
           <add input="{REQUEST_URI}" pattern="~[1-9](?=/|\.)" />
           <add input="{QUERY_STRING}" pattern="~[1-9](?=/|\.)" />
         </conditions>
         <action type="CustomResponse" statusCode="400" statusReason="Bad Request" />
       </rule>
     </rules>
   </rewrite>
   ```
3. **重载配置**  
   ```cmd
   iisreset            :: 整个 IIS 重启，约 2 秒
   ```
> 此后任何含 `~1`…`~9` 的请求直接返回 400，不再产生 404/400 差异 。  

---

## ✅ 验证方式  

| 场景 | 预期结果 |  
|---|---|  
| 浏览器访问 `https://site/*~1*/a.aspx` | **400 Bad Request**（不再是 404） |  
| 命令行 `curl -I https://site/*~1*/.aspx` | `HTTP/1.1 400 Bad Request` |  
| 扫描器（Nessus、IIS-ShortName-Scanner） | 漏洞项降为 **0** |  

> 若仍返回 404，说明该文件短名仍存在→继续执行“目录重建”步骤。  

---

## 🔙 回滚方案  
- **全局重新启用 8.3**：  
  ```cmd
  fsutil 8dot3name set 0
  ```
- **注册表改回 0** 后重启即可恢复生成短名。  

---

## ❓ 常见疑问  

**Q1 关闭 8.3 会影响业务吗？**  
现代 Web/ASP.NET/PHP 均无影响；仅老旧 16-bit 程序或早期 InstallShield 可能无法安装。  

**Q2 不重启能否彻底修复？**  
不能。`fsutil 8dot3name set 1` **必须重启** 才禁止系统卷生成新短名；在线阶段可先用 **URL Rewrite** 把攻击面降到 0。  

**Q3 已上云（虚拟主机）无法改注册表？**  
联系云厂商或在 **web.config** 里部署上述 Rewrite 规则，同样能通过外部扫描。  

---

## 📝 结论  

按 **“关闭 8.3 → 重建目录 → Rewrite 兜底”** 三步落地，可彻底消除 IIS 短文件名泄露风险：  
- **重启前**即可通过 Rewrite 阻断外部枚举；  
- **重启后** NTFS 不再生成新短名，达成永久免疫。  

> 建议在下次计划维护窗口完成重启，形成闭环。