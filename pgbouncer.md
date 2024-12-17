## pgbouncer configuration/settings

# step 1

Get the database credentials from the database container

- Run without the pgbouncer service

```bash
docker-compose up --build -d
```

- Get the database credentials

```bash
docker exec -it chamazetu_webapp_chamazetu_database_1 bash
psql -U mankindjnr -d chamazetudb
```

- Get the database credentials

```sql
SELECT usename, passwd FROM pg_shadow;
```

- Copy the password and username to the pgbouncer userlist.txt file

```txt
"mankindjnr" "SCRAM-SHA-256$4096:OXHwsS64yy6RA1P2k2plDA==$nuA6M10IlDIhUjUcAYB9hs1cXSHJ5mELOY8caF6n0mU=:hyv+t0nExCUJwykY+S6dL80xNWIpyCI4QSge4BjKB4U="
```

# step 2

Bring all services down

```bash
docker-compose down
```

# step 3

Create a pgbouncer.ini file

```ini
[databases]
chamazetudb = host=chamazetu_database port=5432 dbname=chamazetudb user=mankindjnr password=tNNhwY1XOwwQPkhL

[pgbouncer]
listen_addr = 0.0.0.0
listen_port = 6432
auth_type = scram-sha-256
auth_file = /etc/pgbouncer/userlist.txt
admin_users = postgres, mankindjnr
pool_mode = transaction
max_client_conn = 100
default_pool_size = 20
logfile = /var/log/pgbouncer/pgbouncer.log
pidfile = /var/run/pgbouncer/pgbouncer.pid
```

# pgbouncer and services in docker-compose

```yml
chamazetu_database:
  image: postgres:latest
  restart: always
  environment:
    - POSTGRES_USER=${POSTGRES_USER}
    - POSTGRES_PASSWORD={POSTGRES_PASSWORD}
    - POSTGRES_DB=${POSTGRES_DB}
    - POSTGRES_HOST_AUTH_METHOD=scram-sha-256
    - POSTGRES_INITDB_ARGS=--auth=scram-sha-256
  volumes:
    - chamazetu_data:/var/lib/postgresql/data
  ports:
    - "5432:5432"
  networks:
    - chamazetu_network

pgbouncer:
  image: edoburu/pgbouncer:latest
  restart: always
  environment:
    - DB_USER=${POSTGRES_USER}
    - DB_PASSWORD=${POSTGRES_PASSWORD}
    - DB_HOST=chamazetu_database
    - AUTH_TYPE=scram-sha-256
    - POOL_MODE=transaction
    - LISTEN_PORT=6432
  ports:
    - "6432:6432"
  depends_on:
    - chamazetu_database
  volumes:
    - ./pgbouncer.ini:/etc/pgbouncer/pgbouncer.ini
    - ./userlist.txt:/etc/pgbouncer/userlist.txt
  networks:
    - chamazetu_network

chamazetu_backend:
  build:
    context: ./backend_chamazetu
  command: uvicorn app.main:app --host 0.0.0.0 --port 9400 --reload
  volumes:
    - ./backend_chamazetu:/app/backend/
  ports:
    - "9400:9400"
  depends_on:
    - message_broker
    - chamazetu_database
    - pgbouncer
  environment:
    - TZ=Africa/Nairobi
    - DATABASE_URL=postgres://${DB_USER}:${DB_PASSWORD}@pgbouncer:6432/${DB_NAME}
  networks:
    - chamazetu_network

volume:
  chamazetu_data:
```

# step 4

# connect to pgbouncer with psql

```bash
psql -h localhost -p 6432 -U mankindjnr -d chamazetudb
```

# step 5

confirm the files .ini and .txt in the pgbouncer container

```bash
docker exec -it chamazetu_webapp_pgbouncer_1 sh
cd /etc/pgbouncer
cat pgbouncer.ini
cat userlist.txt
```

---

Your service should be up and running

---

# Complete docker-compose file

```yml
services:
  chamazetu_database:
    image: postgres:latest
    restart: always
    environment:
      - POSTGRES_USER=${POSTGRES_USER}
      - POSTGRES_PASSWORD=${POSTGRES_PASSWORD}
      - POSTGRES_DB=${POSTGRES_DB}
      - POSTGRES_HOST_AUTH_METHOD=scram-sha-256
      - POSTGRES_INITDB_ARGS=--auth=scram-sha-256
    volumes:
      - chamazetu_data:/var/lib/postgresql/data
    ports:
      - "5432:5432"
    networks:
      - chamazetu_network

  pgbouncer:
    image: edoburu/pgbouncer:latest
    restart: always
    environment:
      - DB_USER=${POSTGRES_USER}
      - DB_PASSWORD=${POSTGRES_PASSWORD}
      - DB_HOST=chamazetu_database
      - AUTH_TYPE=scram-sha-256
      - POOL_MODE=transaction
      - LISTEN_PORT=6432
    ports:
      - "6432:6432"
    depends_on:
      - chamazetu_database
    volumes:
      - ./pgbouncer.ini:/etc/pgbouncer/pgbouncer.ini
      - ./userlist.txt:/etc/pgbouncer/userlist.txt
    networks:
      - chamazetu_network

  message_broker:
    image: redis:latest
    ports:
      - "6380:6379"
    volumes:
      - redis_data:/data
    networks:
      - chamazetu_network

  chamazetu_frontend_database:
    image: postgres:latest
    restart: always
    environment:
      POSTGRES_USER: ${FRONTEND_POSTGRES_USER}
      POSTGRES_PASSWORD: ${FRONTEND_POSTGRES_PASSWORD}
      POSTGRES_DB: ${FRONTEND_POSTGRES_DB}
    volumes:
      - frontend_data:/var/lib/postgresql/data
    ports:
      - "5434:5432"
    networks:
      - chamazetu_network

  chamazetu_frontend:
    image: ghcr.io/mankindjnr/chamazetu_frontend:latest
    command: >
      sh -c "python manage.py collectstatic --no-input && gunicorn frontend_chamazetu.wsgi:application --bind 0.0.0.0:8000 --workers 3"
    environment:
      - TZ=Africa/Nairobi
    depends_on:
      - chamazetu_frontend_database
      - message_broker
    networks:
      - chamazetu_network
    deploy:
      replicas: 3

  chamazetu_backend:
    image: ghcr.io/mankindjnr/chamazetu_backend:latest
    command: uvicorn app.main:app --host 0.0.0.0 --port 9400 --workers 4
    environment:
      - TZ=Africa/Nairobi
      - DATABASE_URL=postgres://${DB_USER}:${DB_PASSWORD}@pgbouncer:6432/${DB_NAME}
    depends_on:
      - chamazetu_database
      - message_broker
      - pgbouncer
    networks:
      - chamazetu_network
    deploy:
      replicas: 3

  celery_worker:
    image: ghcr.io/mankindjnr/celery_worker:latest
    command: celery -A frontend_chamazetu.celery worker -l INFO
    environment:
      - TZ=Africa/Nairobi
      - DATABASE_URL=postgres://${FRONTEND_POSTGRES_USER}:${FRONTEND_POSTGRES_PASSWORD}@chamazetu_frontend_database:5432/${FRONTEND_POSTGRES_DB}
      - C_FORCE_ROOT=true
    depends_on:
      - message_broker
    networks:
      - chamazetu_network
    deploy:
      replicas: 3

  celery_beat_scheduler:
    image: ghcr.io/mankindjnr/celery_beat_scheduler:latest
    command: celery -A frontend_chamazetu.celery beat -l INFO
    environment:
      - DATABASE_URL=postgres://${FRONTEND_POSTGRES_USER}:${FRONTEND_POSTGRES_PASSWORD}@chamazetu_frontend_database:5432/${FRONTEND_POSTGRES_DB}
      - TZ=Africa/Nairobi
    depends_on:
      - message_broker
    networks:
      - chamazetu_network

  nginx:
    image: nginx:latest
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
      - ./certbot/conf:/etc/letsencrypt
      - ./certbot/www:/var/www/certbot
    ports:
      - "80:80"
      - "443:443"
    depends_on:
      - chamazetu_frontend
      - chamazetu_backend
    networks:
      - chamazetu_network

volumes:
  redis_data:
  chamazetu_data:
  frontend_data:

networks:
  chamazetu_network:
    driver: bridge
```

Remember to have the pgbouncer.ini, .env, nginx.conf and userlist.txt files in the root directory of the project, same level as the docker-compose.yml file and the certbot directory

To restart the pgbouncer service

```bash
docker exec -it chamazetu_webapp_pgbouncer_1 pgbouncer -R -d /etc/pgbouncer/pgbouncer.ini
```

# if you want change postgres password

```bash
docker exec -it chamazetu_webapp_chamazetu_database_1 bash

su - postgres

psql -U mankindjnr -d chamazetudb

ALTER USER mankindjnr WITH PASSWORD 'tNNhwY1XOwwQPkhL';
```

# pgbouncer commands

```bash
docker exec -it chamazetu_webapp_pgbouncer_1 pgbouncer -R -d /etc/pgbouncer/pgbouncer.ini

docker exec -it chamazetu_webapp_pgbouncer_1 pgbouncer -u mankindjnr -d /etc/pgbouncer/pgbouncer.ini

docker exec -it chamazetu_webapp_pgbouncer_1 pgbouncer -u mankindjnr -d /etc/pgbouncer/pgbouncer.ini -R
```

# CONFIGURATION IN PRODUCTION

---

---

user supabase credentials or use supabase inbuild pgbouncer

# errors

---

The error message FATAL pidfile exists, another instance running? indicates that PgBouncer is detecting an existing PID file, which suggests that another instance of PgBouncer might be running or that a previous instance did not shut down cleanly.

To resolve this issue, follow these steps:

Stop the Docker Compose services:

```bash
docker-compose down
```

Remove any existing PID files:

```bash
find . -name "*.pid" -exec rm -f {} \;
```

Restart the Docker Compose services:

```bash
docker-compose up -d
```

This should clean up any leftover PID files and allow PgBouncer to start correctly.
