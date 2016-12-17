---
layout: default
title: Sqlalchemy 使用
description: ''
show_header: true
---


使用alembic和sqlalchemy结合进行数据库迁移的控制。
假设项目目录yourproject
```
cd yourproject
alembic list_templates (罗列可供使用的模板)
alembic init migration(可以其他名称) (初始化)
alembic revision -m "create account table" (创建空的迁移脚本)
alembic revision --autogenerate -m "Added account table" (自动对比数据库，创建迁移脚本)
alembic upgrade head (前进迁移版本)
alembic current (获得现在的版本信息)
alembic downgrade base (回退迁移版本)
```