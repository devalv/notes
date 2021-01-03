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