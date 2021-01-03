1. Подключить Amazon Simple Notifications Service для уведомлений о результатах задач в разделе Notifications по ссылке https://eu-central-1.console.aws.amazon.com/glacier/home?region=eu-central-1#/vaults
1. Установить консольный клиент ```apt install awscli```
1. В каталоге ```~/.aws``` создать 2 файла (credentials и config) ```printf "[default]\nregion=eu-central-1\n" > ~/.aws/config && printf "[default]\naws_access_key_id = qwe\naws_secret_access_key = qwe\n" > ~/.aws/credentials```
1. Список хранилищ ```aws glacier list-vaults --account-id 123456```
1. Список выполняющихся заданий ```aws glacier list-jobs --account-id 123456 --vault-name some-project```
1. Получить список архивов ```aws glacier initiate-job --account-id 123456 --vault-name some-project --job-parameters '{"Type": "inventory-retrieval"}'```
1. ПОСЛЕ того, как задание будет выполнено - получить его результат ```aws glacier get-job-output --account-id 123456 --vault-name some-project --job-id 4yj04OEL5fU output.json```
1. Удаление конкретного архива ```aws glacier delete-archive --account-id 123456 --vault-name some-project --archive-id="0cz0GfrCqI"```
1. Список файлов на S3 (тут хранятся последние актуальные архивы) ```https://s3.console.aws.amazon.com/s3/buckets/some-project/?region=eu-central-1&tab=overview```

Скачать файл с архивом с S3 можно через веб-интерфейс. На S3 хранятся дампы не старше 3 дней.  Более старые версии нужно скачивать из Glacier, согласно инструкции.