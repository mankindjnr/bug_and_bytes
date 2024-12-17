```sql
TRUNCATE TABLE celery_taskmeta RESTART IDENTITY CASCADE;
TRUNCATE TABLE celery_tasksetmeta RESTART IDENTITY CASCADE;
TRUNCATE TABLE periodic_task RESTART IDENTITY CASCADE;
TRUNCATE TABLE periodic_tasks RESTART IDENTITY CASCADE;
TRUNCATE TABLE django_celery_beat_periodictask RESTART IDENTITY CASCADE;
```

# redis

```bash
docker exec -it redis redis-cli
# or
redis-cli -p 6379

# list all keys
keys *

# delete all keys
flushall
```
