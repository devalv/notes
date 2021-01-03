## docker-compose

```
version: '3.7'
services:
  postgres:
    image: postgres:9.6
    ports:
        - 5432:5432
    environment:
      - POSTGRES_DB=postgres
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=postgres
    container_name: postgres
    volumes:
      - ./postgres/:/docker-entrypoint-initdb.d/
  frontend-dev:
    image: teracy/angular-cli
    ports:
      - 4200:4200
    container_name: angular-dev
    volumes:
      - ./frontend:/opt/some-project
    command: sh /opt/some-project/entrypoint.sh -dev
  nginx-proxy:
    image: nginx
    container_name: nginx
    volumes:
      - ./nginx:/opt/some-project
      - ../conf:/opt/some-project/conf
      - ../../frontend/:/opt/some-project/frontend
    command: sh /opt/some-project/entrypoint.sh
    network_mode: host
```

***

## Postgres entry point
```
echo "Adding no password check for existing records in pg_hba.conf"
sed -i 's/peer/trust/g' /var/lib/postgresql/data/pg_hba.conf
sed -i 's/md5/trust/g' /var/lib/postgresql/data/pg_hba.conf

echo "Extending max connections in postgresql.conf"
sed -i 's/max_connections = 100	/max_connections = 1000	/g' /var/lib/postgresql/data/postgresql.conf

echo "Creatind database"
psql -c "create database some-database encoding 'utf8' lc_collate = 'C.UTF-8' lc_ctype = 'C.UTF-8' template template0;" -U postgres

#echo "Running pg_restore"
#pg_restore -d some-database /docker-entrypoint-initdb.d/20191021.tar
```

***

## nginx entry point
```
#!/bin/bash

APP_DIR=/opt/some-project/conf
cd $APP_DIR || exit
echo "Stopping nginx..."
/etc/init.d/nginx stop
echo "Removing default config file..."
rm /etc/nginx/conf.d/default.conf
echo "Copy local config to nginx default dir"
cp nginx-dev.conf /etc/nginx/conf.d/default.conf
echo "Checking nginx config file..."
/etc/init.d/nginx configtest
echo "Starting nginx..."
nginx -g 'daemon off;'
echo "Nginx status is:"
/etc/init.d/nginx status
```

***
## Angular entry point
```
#!/bin/bash

APP_DIR=/opt/some-project/frontend
cd $APP_DIR || exit

usage() {
  cat <<-EOF
  Usage: sh /opt/some-project/entrypoint.sh [options]
  Options:
    -h,    --help                 output help information
    -dev,  --development          install dependencies and run live angular server
    -prod, --production           clean install dependenciec and build angular with production arg
EOF
}

dev() {
  echo "Install dependencies and run live Angular server"
  echo "Directory is:";
  pwd;
  echo "Removing old dist and node_modules:"
  rm -rf node_modules dist;
  echo "Installing dependencies via npm:"
  npm install --unsafe-perm;
  echo "Running angular-live:"
  npm start -- -c=docker --host=0.0.0.0
}

prod() {
  echo "Clean install dependencies and build Angular with production arg";
  echo "Directory is:"
  pwd;
  echo "Removing old dist and node_modules:"
  rm -rf node_modules dist;
  echo "Installing dependencies via npm:"
  npm install --unsafe-perm;
  echo "Building code:"
  npm run build -- --prod
  echo "Code compilation completed. Files are in: frontend/"
}

# parse argv
while test $# -ne 0; do
  arg=$1; shift
  case $arg in
    -h|--help) usage; exit ;;
    -dev|--development) dev; exit ;;
    -prod|--production) prod; exit ;;
  esac
done
```