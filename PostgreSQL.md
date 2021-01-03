## Простая схема резервирования 
([uploader.py](https://github.com/devalv/aws_uploader))

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

## Скрипт резервных копий
[postgrespro-backup.sh](https://github.com/devalv/bash/blob/master/postgrespro-backup.sh)