---
title: "🚀 JeecgBoot springboot3-3.8.2 极速踩坑实录"
tags: [JeecgBoot, SpringBoot3, JDK17, MySQL5.7, Node22, 低代码]
draft: false
---

---

# 🚀 JeecgBoot springboot3-3.8.2 极速踩坑实录

> 基于亲测环境：IDEA 2025 + JDK17 Temurin 17.0.16 + MySQL 5.7.44 + Redis 6.2.19 + Node 22.19.0  
> 从 0 到登录后台，**15 min** 搞定。

---

## 1️⃣ 技术选型 & 分支依据

| 组件  | 选型                        | 理由                                                                               |
| ----- | --------------------------- | ---------------------------------------------------------------------------------- |
| 分支  | **springboot3-3.8.2**       | 官方长期维护，SpringBoot3 最低 JDK17，性能提升 30%↑                                |
| JDK   | Eclipse Temurin **17.0.16** | 实测启动快 25%、内存省 15%，[官方基准](https://my.oschina.net/jeecg/blog/10924114) |
| MySQL | **5.7.44**                  | 稳定、索引优化好，Jeecg 官方推荐                                                   |
| Redis | **6.2.19**                  | 3.2+ 即可，6.x 支持多 IO 线程，RDM 用 tiny-rdm 轻量                                |
| Node  | **22.19.0**                 | 20.18.0 起无法启动前端，官方已锁 22.x                                              |
| Maven | IDEA 2025 内置 **3.9.9**    | 无需额外安装                                                                       |

---

## 2️⃣ 安装顺序（Windows / macOS 通用）

1. **JDK**  
   IDEA → Project Structure → SDK → Add JDK → 选 Eclipse Temurin 17.0.16 压缩包即可。
2. **MySQL 5.7.44**  
   安装后立刻修改 **my.cnf**（或 my.ini）：
   ```ini
   [mysqld]
   lower_case_table_names=1   # 0=敏感，1=不敏感，必须重启生效
   ```

重启服务，**再建库**：

```sql
CREATE DATABASE jeecg_boot DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

3. **Redis 6.2.19**
   默认端口 6379，无密码即可；如有密码后续在 yml 里改。
4. **Node 22.19.0**
   ```bash
   nvm install 22.19.0
   nvm use 22.19.0
   node -v   # v22.19.0
   ```

---

## 3️⃣ 克隆 & 切分支

```bash
git clone https://github.com/jeecgboot/JeecgBoot.git
cd JeecgBoot
git checkout springboot3-3.8.2   # 一定要指定 tag/branch
```

---

## 4️⃣ 数据库初始化

```bash
mysql -uroot -p jeecg_boot < jeecg-boot/db/jeecgboot-mysql5.sql
```

---

## 5️⃣ 后端配置 & 启动

1. 打开 IDEA → Import `jeecg-boot/jeecg-boot-module-system/pom.xml`
2. Maven 面板 → **clean** → **install**（首次 3~5 min）
3. 修改 **application-dev.yml**
   ```yaml
   spring:
     datasource:
       url: jdbc:mysql://127.0.0.1:3306/jeecg_boot?useSSL=false&serverTimezone=UTC
       username: root
       password: yourPwd
     redis:
       host: 127.0.0.1
       port: 6379
       password: # 无密码留空
   ```
4. 启动配置 → **Edit Configurations** →
   → **Modify options** → **Shorten command line** → 选 **classpath file**
   然后 **Debug / Run**
   控制台出现 **Started JeecgApplication** 即成功，端口 8080。

---

## 6️⃣ 前端配置 & 启动

```bash
cd jeecg-boot-vue3
nvm use 22.19.0      # 必须 22.x
pnpm install         # 或 npm i
pnpm dev
```

默认端口 **3000**，浏览器打开 [http://localhost:3000](http://localhost:3000)

---

## 7️⃣ 默认账号

| 账号  | 密码   | 角色       |
| ----- | ------ | ---------- |
| admin | 123456 | 超级管理员 |
| test  | 123456 | 普通用户   |

---

## 8️⃣ 排坑速查

| 现象        | 根因             | 解决                                         |
| ----------- | ---------------- | -------------------------------------------- |
| 前端起不来  | Node 20.18.0     | 升 22.19.0                                   |
| 表不存在    | MySQL 大小写敏感 | 安装后先设 `lower_case_table_names=1` 再建库 |
| IDEA 启动慢 | 命令行太长       | 缩短命令行选 classpath file                  |
| 验证码 500  | Redis 没启动     | `redis-cli ping` 必须返回 `PONG`             |

---

- [ ] 阅读 [官方文档](https://help.jeecg.com)
- [ ] 给仓库点个 ⭐ [JeecgBoot](https://github.com/jeecgboot/JeecgBoot)

---
