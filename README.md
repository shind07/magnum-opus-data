# magnum-opus-data
Data infra for Magnum Opus

# Journal
- Going to use the player stats table: https://github.com/nflverse/nflverse-data/releases/download/player_stats/player_stats.csv
- Getting the data schema by running `csvsql -i sqlite data/player_stats.csv`
- Using this as a template for my sqlite docker implementation https://github.com/nouchka/docker-sqlite3/blob/master/README.md
- Initalize the db with `make init`
- Populate the db with `python3 app/load.py`

## Setting up the DB

### Credentials
https://hub.docker.com/_/mysql

Environment variables:
- `MYSQL_ROOT_PASSWORD` - REQUIRED, sets password for root user
- `MYSQL_DATABASE` - Optional, name of DB to create on image startup
- `MYSQL_USER`, `MYSQL_PASSWORD` - Optional, but required together. User will be granted superuser permissions for `MYSQL_DATABASE`.

### .env
- Docker will load the credentials mentioned above from a .env file. Example:
```
MYSQL_ROOT_PASSWORD=password
MYSQL_DATABASE=nfl
```
- Load the .env file into the db service:
```yml
services:
  db:
    env_file:
      - .env
```

### Initialization
- Upon starting the container for the first time, any files in `docker-entrypoint-initdb.d` with extensions .sh, .sql and .sql.gz will be executed (in alphabetical order).

### Data Persistence
- We want to store our data in a volume mount so it persists when the image/service stops.
- Specify a volumne in `docker-compose.yml`:
```yml
volumes:
  db-data:
```
- Attach that volume to the db service:
```yaml
services:
  db:
    image: mysql:8
    container_name: nfl-db
    volumes:
      - ./data/db/init.sql:/docker-entrypoint-initdb.d/init.sql
      - db-data:/var/lib/mysql
```
- The volume must be mounted to `/var/lib/mysql`.

## Resources

https://docs.docker.com/samples/flask/
https://github.com/nflverse/nflverse-data
https://hub.docker.com/_/mysql
https://dev.mysql.com/doc/mysql-getting-started/en/