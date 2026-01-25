### 缓存
```sql
-- create table
CREATE UNLOGGED TABLE cache (
  key TEXT PRIMARY KEY,
  value JSONB NOT NULL,
  expires_at TIMESTAMPTZ NOT NULL
);

CREATE INDEX idx_cache_expires ON cache(expires_at);

-- insert
INSERT INTO cache (key, value, expires_at)
VALUES ($1, $2, NOW() + INTERVAL '1 hour')
ON CONFLICT (key) DO UPDATE
  SET value = EXCLUDED.value,
      expires_at = EXCLUDED.expires_at;

-- read
SELECT value FROM cache
WHERE key = $1 AND expires_at > NOW();

-- cleanup
DELETE FROM cache WHERE expires_at < NOW();
```

### 订阅
```sql
NOTIFY notifications, '{"userId": 123, "msg": "Hello"}';

LISTEN notifications;
```

### 任务队列
```sql
-- create table
CREATE TABLE jobs (
  id BIGSERIAL PRIMARY KEY,
  queue TEXT NOT NULL,
  payload JSONB NOT NULL,
  attempts INT DEFAULT 0,
  max_attempts INT DEFAULT 3,
  scheduled_at TIMESTAMPTZ DEFAULT NOW(),
  created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_jobs_queue ON jobs(queue, scheduled_at) 
WHERE attempts < max_attempts;

-- insert 
INSERT INTO jobs (queue, payload)
VALUES ('send-email', '{"to": "user@example.com", "subject": "Hi"}');

-- select
WITH next_job AS (
  SELECT id FROM jobs
  WHERE queue = $1
    AND attempts < max_attempts
    AND scheduled_at <= NOW()
  ORDER BY scheduled_at
  LIMIT 1
  FOR UPDATE SKIP LOCKED
)
UPDATE jobs
SET attempts = attempts + 1
FROM next_job
WHERE jobs.id = next_job.id
RETURNING *;
```

### 速率限制
```sql
CREATE TABLE rate_limits (
  user_id INT PRIMARY KEY,
  request_count INT DEFAULT 0,
  window_start TIMESTAMPTZ DEFAULT NOW()
);

-- Check and increment
WITH current AS (
  SELECT 
    request_count,
    CASE 
      WHEN window_start < NOW() - INTERVAL '1 minute'
      THEN 1  -- Reset counter
      ELSE request_count + 1
    END AS new_count
  FROM rate_limits
  WHERE user_id = $1
  FOR UPDATE
)
UPDATE rate_limits
SET 
  request_count = (SELECT new_count FROM current),
  window_start = CASE
    WHEN window_start < NOW() - INTERVAL '1 minute'
    THEN NOW()
    ELSE window_start
  END
WHERE user_id = $1
RETURNING request_count;

-- window function
CREATE TABLE api_requests (
  user_id INT NOT NULL,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Check rate limit
SELECT COUNT(*) FROM api_requests
WHERE user_id = $1
  AND created_at > NOW() - INTERVAL '1 minute';

-- If under limit, insert
INSERT INTO api_requests (user_id) VALUES ($1);

-- Cleanup old requests periodically
DELETE FROM api_requests WHERE created_at < NOW() - INTERVAL '5 minutes';
```

### 用户会话
```sql
CREATE TABLE sessions (
  id TEXT PRIMARY KEY,
  data JSONB NOT NULL,
  expires_at TIMESTAMPTZ NOT NULL
);

CREATE INDEX idx_sessions_expires ON sessions(expires_at);

-- Insert/Update
INSERT INTO sessions (id, data, expires_at)
VALUES ($1, $2, NOW() + INTERVAL '24 hours')
ON CONFLICT (id) DO UPDATE
  SET data = EXCLUDED.data,
      expires_at = EXCLUDED.expires_at;

-- Read
SELECT data FROM sessions
WHERE id = $1 AND expires_at > NOW();
```

来源
[I Replaced Redis with PostgreSQL (And It's Faster)](https://dev.to/polliog/i-replaced-redis-with-postgresql-and-its-faster-4942)
