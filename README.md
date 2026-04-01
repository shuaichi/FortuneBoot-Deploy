# 部署说明

## Docker 部署（推荐）

项目提供预构建的 Native Image Docker 镜像，支持 `linux/amd64` 和 `linux/arm64` 架构。

镜像仓库：
- DockerHub：`shuaichi/fortuneboot`
- 阿里云：`registry.cn-hangzhou.aliyuncs.com/chishenjianglin/fortuneboot`

## SQLite模式
数据持久化在 `/data` 卷中
```bash
docker run -d \
  --name fortuneboot \
  -p 8080:8080 \
  -v fortuneboot-data:/data \
  shuaichi/fortuneboot:latest
```
或者使用docker-compose启动容器（和docker run模式二选一）
```yaml
services:
    fortuneboot:
        image: 'shuaichi/fortuneboot:latest'
        volumes:
            - 'fortuneboot-data:/data'
        ports:
            - '8080:8080'
        container_name: fortuneboot
```
## MySQL/MariaDB模式
先创建数据库：
```sql
CREATE DATABASE IF NOT EXISTS fortune_boot CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```
然后启动docker容器
```bash
docker run -d \
  --name fortuneboot \
  -p 8080:8080 \
  -e DB_TYPE=mysql \
  -e DB_HOST=your-mysql-host \
  -e DB_PORT=3306 \
  -e DB_NAME=fortune_boot \
  -e DB_USERNAME=root \
  -e DB_PASSWORD=your_password \
  shuaichi/fortuneboot:latest
```
或者使用docker-compose启动容器（和docker run模式二选一）
```yaml
services:
    fortuneboot:
        image: 'shuaichi/fortuneboot:latest'
        environment:
            - DB_PASSWORD=your_password
            - DB_USERNAME=root
            - DB_NAME=fortune_boot
            - DB_PORT=3306
            - DB_HOST=your-mysql-host
            - DB_TYPE=mysql
        ports:
            - '8080:8080'
        container_name: fortuneboot
```

---

# 配置说明

## 关键配置项

| 配置项 | 说明 | 默认值                            |
|---|---|--------------------------------|
| `server.port` | 服务端口 | `8080`                         |
| `db.type` | 数据库类型（`mysql` / `sqlite`） | `sqlite`                       |
| `db.sqlite.path` | SQLite 数据库文件路径 | `/data/fortuneboot.db`         |
| `fortuneboot.api-prefix` | API 请求前缀（用于前端代理转发） | `/dev-api`（开发）/ `/prod-api`（生产） |
| `token.secret` | JWT 密钥 | 内置默认值                |
| `token.autoRefreshTime` | Token 自动刷新时间（分钟） | `21600`（15天）                   |
## 环境变量支持

以下配置支持通过环境变量覆盖：

| 环境变量 | 对应配置                |
|---|---------------------|
| `DB_TYPE` | 数据库类型               |
| `DB_HOST` | MySQL 主机地址          |
| `DB_PORT` | MySQL 端口            |
| `DB_NAME` | 数据库名称               |
| `DB_USERNAME` | 数据库用户名              |
| `DB_PASSWORD` | 数据库密码               |
| `DB_PATH` | SQLite 文件路径         |
| `DRUID_USERNAME` | 德鲁伊数据库连接池帐号         |
| `DRUID_PASSWORD` | 德鲁伊数据库连接池密码         |
| `TOKEN_SECRET` | JWT 密钥（生产环境务必更换）    |
| `RSA_PRIVATE_KEY` | RSA 私钥（生产环境务必更换）    |
| `RSA_PUBLIC_KEY` | RSA 公钥（生产环境务必更换）    |
| `SWAGGER_ENABLE` | Swagger UI 开关（生产环境） |

---

## 数据库初始化

### 自动迁移（推荐）

项目集成了 Flyway 数据库版本迁移工具，**首次启动时会自动创建表结构并初始化数据**，无需手动执行 SQL。

- **MySQL 模式**：迁移脚本位于 `fortuneboot-infrastructure/src/main/resources/db/migration/mysql/`，包含从 V1.0.0 到 V1.5.0 的完整增量迁移链
- **SQLite 模式**：迁移脚本位于 `fortuneboot-infrastructure/src/main/resources/db/migration/sqlite/`，使用 V1.5.0 全量脚本

对于已有数据的 MySQL 实例，Flyway 会通过表/列存在性智能检测当前版本并设置 baseline，无需担心数据丢失。

# APP下载地址
支持安卓和IOS，欢迎来试用。

https://www.fortuneboot.com/archives/hao-ji-appxia-zai

# 9快记账数据迁移至好记记账

1、好记数据库依赖mysql8.0及以上版本，如果你是使用的5.7请先升级mysql版本后再升级。

2、在9快数据库实例上创建一个新的schema，并执行上面创建数据库表的sql。

3、下载数据迁移脚本:https://github.com/shuaichi/docker-compose-ali/blob/main/9%E5%BF%AB%E6%95%B0%E6%8D%AE%E8%BF%81%E7%A7%BB%E8%87%B3%E5%A5%BD%E8%AE%B0.sql

4、修改如下数据库脚本至自己的。

```sql
  -- 使用新的数据库
  use fortune_boot;
  -- 新的数据库
  SET @new_schema = 'fortune_boot';
  -- 旧的数据库
  SET @old_schema = 'moneynote';
  -- 迁移到好记的用户 id
  SET @new_user_id = 1;
  -- 要迁移的用户id
  SET @old_user_id = 1;
```

5、执行数据库脚本。

---

# 更新日志

| 版本 | 主要变更 |
|---|---|
| **v1.5.0** | 去除 Redis 依赖，登录令牌持久化到数据库（`sys_login_token` 表） |
| **v1.4.0** | 角色和用户新增超级管理员标识字段 |
| **v1.3.0** | 新增单据管理功能（报销单），账单支持关联单据 |
| **v1.2.0** | 新增周期记账功能（Quartz 定时任务 + CRON 表达式），支持执行日志记录 |
| **v1.1.6** | 新增首页大屏金额显示/隐藏配置项 |
| **v1.1.0** | 新增归物（GoodsKeeper）功能，计算物品持有成本 |
| **v1.0.0** | 基础版本：用户管理、角色管理、菜单管理、账户管理、账本管理、分类/标签/交易对象管理、账单管理 |

---

## 交流

QQ 群：[![加入QQ群](https://img.shields.io/badge/1009576058-blue.svg)](https://qm.qq.com/q/M2zyt7vxyG)

如果觉得项目对您有帮助，欢迎 Star 支持。

---

## 许可证

[MIT License](LICENSE)