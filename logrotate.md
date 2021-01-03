## Пример удобного конфига ротации логов

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