# notes
Some useful notes

## Git

### Undo

Delete the most recent commit, keeping the work you've done:
```
git reset --soft HEAD~1
```
Delete the most recent commit, **destroying the work** you've done:
```
git reset --hard HEAD~1
```

### Git hooks

1. Установить flake8 и плагины
    ```pip install flake8 flake8-builtins flake8-bugbear```

2. Установить хук для Git:
    ```flake8 --install-hook git```

3. Настроить Git, чтобы он учитывал правила Flake8:
    ```git config --bool flake8.strict true```
    
### Git flow config
[Описание](https://danielkummer.github.io/git-flow-cheatsheet/index.ru_RU.html)
```
[gitflow "branch"]
    master = master
    develop = develop
[gitflow "prefix"]
    feature = feature_tg_
    bugfix = fix_tg_
    release = release_
    hotfix = hf_tg_
    support = support/
    versiontag = release-
[gitflow "path"]
    hooks = /home/user/PycharmProjects/some-project/.git/hooks
```

### Github Actions
Пример автозапуска тестов при push и pull_request в дев и мастер.

```
# Runs tests and coverage measuring.
name: tests

on:
  push:
    branches: [ master, develop ]
  pull_request:
    branches: [ master, develop ]

jobs:
  build:

    runs-on: ubuntu-latest
    strategy:
      matrix:
        python-version: [3.5]

    steps:
    - uses: actions/checkout@v2
    - name: Set up Python ${{ matrix.python-version }}
      uses: actions/setup-python@v1
      with:
        python-version: ${{ matrix.python-version }}
    - name: Install dependencies
      run: |
        python -m pip install --upgrade pip
        python -m pip install pipenv
        pip install flake8
        if [ -f requirements.txt ]; then pip install -r requirements.txt; fi
        if [ -f Pipfile ]; then pipenv install --dev; fi
    - name: Lint with flake8
      run: |
        # stop the build if there are Python syntax errors or undefined names
        pipenv run flake8 . --count --select=E9,F63,F7,F82 --show-source --statistics
        # exit-zero treats all errors as warnings. The GitHub editor is 127 chars wide
        pipenv run flake8 . --count --exit-zero --max-complexity=10 --max-line-length=127 --statistics
    - name: Test with unittest and build coverage xml report
      run: |
        pipenv run coverage run -m unittest discover tests/ && pipenv run coverage xml
    - name: Upload coverage to Codecov
      uses: codecov/codecov-action@v1
      with:
        flags: unittests
        env_vars: OS,PYTHON
        name: codecov-umbrella
        fail_ci_if_error: true
```

## PostgreSQL

Очень простая схема резервирования ([uploader.py](https://github.com/devalv/aws_uploader))
```
#!/bin/bash
now=$(date +"%Y_%m_%d_%HH")'.back.7z'
password='te'$(date +"%Y%m%d")'st'

cd /data/backup
sudo -u postgres pg_dumpall | 7z a -bsp1 -si -p$password $now

if [ $? -eq 0 ]
then
    echo "Dump created successfully. Running uploader"
    /usr/bin/env python3 uploader.py
    if [ $? -eq 0 ]
    then
        echo "Dump uploaded successfully. Everithing is ok"
    else
        echo "Can't upload dump to AWS" >&2
    fi
else
    echo "Could not create dump" >&2
fi
```

[Хороший скрипт резервных копий](https://github.com/devalv/bash/blob/master/postgrespro-backup.sh)

## logrotate

Пример удобного конфига ротации логов
```
# daily, monthly  # сериод ротации
# rotate 2  # сколько старых логов нужно хранить
# copytruncate  # Копирует содержимое файла в новый и очищает содержимое существующего
# dateext  # добавляет дату ротации перед заголовком старого лога
# dateyesterday # использовать вчерашнюю дату при ротации
# compress  # лог необходимо сжимать
# missingok  # не выдавать ошибки, если лог файла не существует;
# su root root # выполнять ротацию логов под указанным пользователем и группой вместо пользователя

/var/log/some-project/*.log {
    daily
    missingok
    rotate 30
    compress
    delaycompress
    copytruncate
    dateext
    dateyesterday
}
```

## nginx

Если нет автопродления или нормального SSL. Если есть - принцип тот же, но все запросы с 80 порта редиректим на 443
 и основной конфиг прописываем там.
```
server {

  listen 80;
  listen 443 ssl;

  client_max_body_size 1M;
  charset utf-8;
  gzip on;

  access_log /var/log/nginx/some-project.acc;
  error_log /var/log/nginx/some-project.err;

  ssl_certificate     /opt/some-project/ssl/default.crt;
  ssl_certificate_key /opt/some-project/ssl/default.key;

  proxy_read_timeout          180s;
  proxy_connect_timeout       180s;
  proxy_send_timeout          180s;
  proxy_redirect              off;
  proxy_http_version          1.1;

  proxy_set_header Upgrade    $http_upgrade;
  proxy_set_header Connection $http_connection;
  proxy_set_header Host       $host;
  proxy_set_header X-Real-IP  $remote_addr;
  proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

  # pass requests for dynamic content to tornado
  location /internal-api/ {
    proxy_pass  http://127.0.0.1:8888/;
  }

  # serve static files
  location / {
    root /opt/some-project/frontend;
    try_files $uri$args $uri$args/ /index.html;
    expires 7d;
  }
}
```

## supervisor

Конфиг супервизора для Tornado (форки внутри процесса)
```
# -*- conf -*-

[program:some-project-8888]
command=/usr/local/bin/pipenv run python app.py --access_to_stdout=True --logging=debug --port=8888 --debug=True --autoreload=False --log_file_prefix=/var/log/some-project/tornado.log  --workers=0
process_name=%(program_name)s
directory=/opt/some-project/backend
environment=PYTHONPATH=/opt/some-project/backend
autorestart=unexpected
autostart=true
startretries=3
stopsignal=TERM
stopasgroup=true
killasgroup=true
```

## Docker

### docker-compose
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

### Postgres entry point
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

### nginx entry point
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

### Angular entry point
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

## SSL
```
acme.sh --upgrade
```

*В конфиге прописать ключ DNS*
```
acme.sh --issue --dns dns_selectel -d devyatkin.dev -d '*.devyatkin.dev'
acme.sh --install-cert -d devyatkin.dev --key-file /opt/some-project/ssl/key.pem  --fullchain-file /opt/some-project/ssl/cert.pem --reloadcmd "service nginx force-reload"
```
[Репозиторий автора](https://github.com/acmesh-official/acme.sh)
[Версия с которой работало](https://github.com/devalv/bash/blob/master/acme.sh)

## AWS

1. Подключить Amazon Simple Notifications Service для уведомлений о результатах задач в разделе Notifications по ссылке https://eu-central-1.console.aws.amazon.com/glacier/home?region=eu-central-1#/vaults
2. Установить консольный клиент ```apt install awscli```
3. В каталоге ```~/.aws``` создать 2 файла (credentials и config) ```printf "[default]\nregion=eu-central-1\n" > ~/.aws/config && printf "[default]\naws_access_key_id = qwe\naws_secret_access_key = qwe\n" > ~/.aws/credentials```
4. Список хранилищ ```aws glacier list-vaults --account-id 123456```
5. Список выполняющихся заданий ```aws glacier list-jobs --account-id 123456 --vault-name some-project```
6. Получить список архивов ```aws glacier initiate-job --account-id 123456 --vault-name some-project --job-parameters '{"Type": "inventory-retrieval"}'```
7. ПОСЛЕ того, как задание будет выполнено - получить его результат ```aws glacier get-job-output --account-id 123456 --vault-name some-project --job-id 4yj04OEL5fU output.json```
8. Удаление конкретного архива ```aws glacier delete-archive --account-id 123456 --vault-name some-project --archive-id="0cz0GfrCqI"```
9. Список файлов на S3 (тут хранятся последние актуальные архивы) ```https://s3.console.aws.amazon.com/s3/buckets/some-project/?region=eu-central-1&tab=overview```

Скачать файл с архивом с S3 можно через веб-интерфейс. На S3 хранятся дампы не старше 3 дней.  Более старые версии нужно скачивать из Glacier, согласно инструкции.

## Python

### Alembic
применить все миграции ```alembic upgrade head```

создание новой миграции ```alembic revision --autogenerate -m "migration text"```

создание пустой миграции ```alembic revision -m "create account table"```

мердж миграций ```alembic merge heads```

### pytest

#### Запуск flake8, прогон тестов pytest и построение отчета coverage

**pytest.ini**
```
[pytest]
addopts = -v -ra --flake8 --cov=<project_name> --cov-report=html
testpaths = tests/
markers =
    config
flake8-ignore =
    E501
    .git/*.* ALL
    __pycache__/*.* ALL
    tests/*.* ALL
```
запуск: `pytest`

### coverage
Генерация отчета в формате html в файле htmlcov/index.html

```
cd $PYTHONPATH && pytest tests/ --cov-report html --cov 
```

или 
```
python3 -m coverage run -m unittest discover tests/ && python3 -m coverage html
```

### pipenv

установить dev-зависимости из Pipfile
```
pipenv install --dev
```

пакеты из pipenv в requirements для pip
```
pipenv lock --requirements > requirements.txt
```

dev-пакеты из pipenv в requirements для pip
```
pipenv lock --dev --requirements > dev_requirements.txt
```

активировать env 
```
pipenv shell
```

### codecov badge
https://github.com/codecov/example-python

### Пакетирование Python

#### PyPI
https://packaging.python.org/tutorials/packaging-projects/

Сборка пакета
```
python3 setup.py sdist bdist_wheel
```

#### deb-пакеты

##### Актуализировать control для пакета
`nano DEBIAN/control`

##### Запустить pyinstaller
```
# Добавить каталог dir и file.txt. Не собирать в 1 файл
pyinstaller app.py -n binary_name --add-data "dir:dir" --add-data "file.txt:."

# Добавить каталог dir и собрать в 1 файл
pyinstaller app.py -n binary_name --add-data "dir:dir" --add-data "file.txt:." -F
```

##### Подготовить итоговую структуру
```
# Сценарий для /usr/bin
mkdir -p binary_name/usr/bin && cp -r dist/* binary_name/usr/bin && cp -r DEBIAN/ binary_name/

# Сценарий для установки в /opt
mkdir -p binary_name/opt/binary_name && cp -r dist/* binary_name/opt && cp -r DEBIAN/ binary_name/
```

##### Собрать итоговый пакет
`dpkg-deb -b binary_name`

## Misc

### Permanent disable fn-mode for Keychron K2 in Ubuntu
1. If this returns "hid_apple", you are using hid_apple ```ls /sys/module | grep hid_apple```
2. ```echo 2 | sudo tee -a /sys/module/hid_apple/parameters/fnmode```
3. Add the option for the fn key ```echo options hid_apple fnmode=2 | sudo tee -a /etc/modprobe.d/hid_apple.conf```
4. Update initramfs bootfile ```sudo update-initramfs -u -k all```
5. Reboot to test (optional)
6. fn + x + l

### Удалить запись из EFI boot loader
1. Run a cmd.exe process with administrator privileges
2. Run `diskpart`
    + Type: list disk then sel disk X where X is the drive your boot files reside on
    + Type list vol to see all partitions (volumes) on the disk (the EFI volume will be formatted in FAT, others will be NTFS)
    + Select the EFI volume by typing: sel vol Y where Y is the SYSTEM volume (this is almost always the EFI partition)
    + For convenience, assign a drive letter by typing: assign letter=Z: where Z is a free (unused) drive letter
    + Type exit to leave disk part

3. While still in the cmd prompt, type: Z: and hit enter, where Z was the drive letter you just created.
    + Type dir to list directories on this mounted EFI partition
    + If you are in the right place, you should see a directory called EFI
    + Type cd EFI and then dir to list the child directories inside EFI
    + Type rmdir /S ubuntu to delete the ubuntu boot directory

4. Install efibootmgr `sudo apt-get install efibootmgr`
5. `sudo modprobe efivars`
6. `sudo efibootmgr`
7. `sudo efibootmgr -b 5 -B`

### Terminal

#### Kill process by name
```
pkill python
```

#### Kill process by pid
```
kill 9476
```

#### Search process by port
```
lsof -i -P -n | grep LISTEN | grep 8888
```

#### redirect 80 to 8888 via iptables
```
sudo iptables -t nat -A PREROUTING -p tcp --dport 80 -j REDIRECT --to-port 8888 && sudo iptables -t nat -I OUTPUT -p tcp -d 127.0.0.1 --dport 80 -j REDIRECT --to-ports 8888
```

#### Multi-ping
```
#!/bin/bash
for i in {1..100}
do
	bash -c "ping 10.55.56.110 -i 0.200 -c 1000 -s 1000;" &
done
```

#### Rsync
```
rsync -ave "ssh -p 5022" user@10.10.0.2:/mnt/media/some-project/logs/old_logs/ /home/user/backups/logs

rsync --remove-source-files -m --include='*.back.7z' --exclude='*' -ave "ssh -p 5022" user@10.10.0.2:/data/backup/ /home/user/backups/Database

rsync --delete -ave "ssh -p 5022" user@10.10.0.2:/mnt/media /home/user/backups/Media
```

#### Redis-cli
```
redis-cli -h 10.10.1.235 -p 6379 -a SeCrEt
```

#### Dates

Current unix-time
```
date +%s
```

Date in format 202004091032
```
date +"%Y%m%d%H%M"
```