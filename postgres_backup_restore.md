docker volume ls
docker volume rm <volume_name>
docker volume prune
docker volume rm -f <volume_name>

- to avoid prompt for password
emacs ~/.pgpass
```bash
localhost:5432:chamazetudb:mankindjnr:tNNhwY1XOwwQPkhL
```

- ## pg_restore/pg_dump should match the version of the postgress db
i.e 17 versio
```bash
sudo apt-get install wget ca-certificates
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
echo "deb http://apt.postgresql.org/pub/repos/apt/ $(lsb_release -cs)-pgdg main" | sudo tee /etc/apt/sources.list.d/pgdg.list
sudo apt-get update
sudo apt-get install postgresql-client-17
pg_dump --version
```
- You can now run the dump or restore commands provided the postgres version is 17

pg_dump -h localhost -p 5432 -U mankindjnr -d chamazetudb -Fc > chamazetu_db_backup_latest.dump

pg_restore -h localhost -p 5432 -U mankindjnr -d chamazetudb ../chamazetu_db_backup_latest.dump