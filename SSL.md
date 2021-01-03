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