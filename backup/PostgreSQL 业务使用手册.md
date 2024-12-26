### 手动修改联合主键表索引

原本 `ad_advertisers` 表以 `app_id`, `channel_name`, `account_id` 三个字段为主键，现在如何往中间再加一个主键字段

```sql
CREATE UNIQUE INDEX ad_advertisers_pkey ON public.ad_advertisers USING btree (app_id, channel_name, account_id);

=>

ALTER TABLE public.ad_advertisers DROP CONSTRAINT ad_advertisers_pkey;
UPDATE ad_advertisers SET account_type = '' WHERE account_type IS NULL;
ALTER TABLE public.ad_advertisers ADD CONSTRAINT ad_advertisers_pkey PRIMARY KEY (app_id, channel_name, account_type, account_id);
```