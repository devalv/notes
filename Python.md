## Alembic
применить все миграции ```alembic upgrade head```

создание новой миграции ```alembic revision --autogenerate -m "migration text"```

создание пустой миграции ```alembic revision -m "create account table"```

мердж миграций ```alembic merge heads```

***

## pytest

### Запуск flake8, прогон тестов pytest и построение отчета coverage

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

## coverage

Генерация отчета в формате html в файле htmlcov/index.html

```
cd $PYTHONPATH && pytest tests/ --cov-report html --cov 
```

или 
```
python3 -m coverage run -m unittest discover tests/ && python3 -m coverage html
```

## pipenv

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

***

## codecov badge
https://github.com/codecov/example-python

***

## Пакетирование Python

### PyPI
https://packaging.python.org/tutorials/packaging-projects/

Сборка пакета
```
python3 setup.py sdist bdist_wheel
```

### deb-пакеты

#### Актуализировать control для пакета
`nano DEBIAN/control`

#### Запустить pyinstaller
```
# Добавить каталог dir и file.txt. Не собирать в 1 файл
pyinstaller app.py -n binary_name --add-data "dir:dir" --add-data "file.txt:."

# Добавить каталог dir и собрать в 1 файл
pyinstaller app.py -n binary_name --add-data "dir:dir" --add-data "file.txt:." -F
```

#### Подготовить итоговую структуру
```
# Сценарий для /usr/bin
mkdir -p binary_name/usr/bin && cp -r dist/* binary_name/usr/bin && cp -r DEBIAN/ binary_name/

# Сценарий для установки в /opt
mkdir -p binary_name/opt/binary_name && cp -r dist/* binary_name/opt && cp -r DEBIAN/ binary_name/
```

#### Собрать итоговый пакет
`dpkg-deb -b binary_name`
