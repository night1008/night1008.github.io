使用 PostgreSQL + PostgREST，可以省去大部分后端编写工作。

1. 复制以下 `docker-compose.yml`

2. 执行 `docker compose up -d`

3. 管理数据库，例如创建表，插入测试数据等
    >访问 http://localhost:8001

4. 使用 API 替代 ORM 等操作
    > API 地址是 http://localhost:8002

5. API 文档地址：
    > https://postgrest-docs-chinese.readthedocs.io/zh/latest/api.html

6. 千万不要暴露 `数据库端口` 到公网

---

> docker-compose.yml

```yaml
version: '3'

services:
  server:
    image: postgrest/postgrest
    ports:
      - "8002:8002"
    environment:
      # 使用 postgres 超级用户
      PGRST_DB_URI: postgres://postgres:your_password@db:5432/app_db
      PGRST_DB_SCHEMA: public
      PGRST_DB_ANON_ROLE: postgres  # 使用超级用户作为匿名角色
      PGRST_OPENAPI_SERVER_PROXY_URI: http://127.0.0.1:8002
      # 允许所有跨域请求
      PGRST_SERVER_PORT: 8002
      PGRST_SERVER_CORS_ALLOWED_ORIGINS: ""
      PGRST_DB_POOL: 50
    depends_on:
      - db
 
  db:
    image: postgres:17.2
    restart: always
    shm_size: 128mb
    environment:
      POSTGRES_DB: app_db
      POSTGRES_USER: postgres  # 使用默认的 postgres 超级用户
      POSTGRES_PASSWORD: your_password
    ports:
      - "5432:5432"
    volumes:
      - pgdata:/var/lib/postgresql/data

  adminer:
    image: adminer
    restart: always
    ports:
      - 8001:8080

volumes:
  pgdata:
```