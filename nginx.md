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