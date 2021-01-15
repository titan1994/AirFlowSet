#Настройка AirFlow Linux+Postgres
После установки по ссылке:

https://airflow.apache.org/docs/apache-airflow/stable/installation.html

Нас ждут весёлые старты, описанные далее. Без этого не взлетит

##________Нельзя без этих инструментов. Редакторы кода:
```
sudo snap install pycharm --classic
sudo snap install sublime-text --classic
```

##________Postgres для AirFlow:

```
sudo apt-get install postgresql
sudo apt-get install build-essential libssl-dev libffi-dev python3-dev
sudo apt-get install libpq-dev

sudo -i -u postgres

create database airflow;
CREATE ROLE admin WITH SUPERUSER CREATEDB CREATEROLE LOGIN ENCRYPTED PASSWORD 'admin';
grant all privileges on database airflow to admin;

\q
exit
```

##________Заходим в окружение AirFlow. Ставим пакеты 
внутри папки my_airflow набрать сначала pipenv shell - чтобы войти в виртуальное окружение проекта

```
pipenv shell 

pip install psycopg2
pip install 'apache-airflow [postgres]'
```

Плагин для редактировнания кода (заработает когда будет создан первый DAG в папке airflow/dags)
```
pip install airflow-code-editor
```

##________Правим airflow.cfg 

экзекьютор меняем на такой:
```
executor = LocalExecutor
```
а также коннектор к базе прописываем на постгрес:
```
sql_alchemy_conn = postgresql+psycopg2://admin:admin@127.0.0.1:5432/airflow
```

##________Правим webserver_config.py(раскомментируем как тут 4 строки)
```
# Uncomment to setup Full admin role name
AUTH_ROLE_ADMIN = 'Admin'

# Uncomment to setup Public role name, no authentication needed
AUTH_ROLE_PUBLIC = 'Public'

# Will allow user self registration
AUTH_USER_REGISTRATION = True

# The default user self registration role
AUTH_USER_REGISTRATION_ROLE = "Public"
```

##________Создаем папку airflow/dags
просто создаем папку 

##________ЗАПУСК шаг -1. Инициировать базу
Инит базы данных. Один раз достаточно. 
Либо каждый раз при перенастройке базы данных

```
airflow db init
```

##________ЗАПУСК шаг0. Создаем Адмнинистратора

```
airflow users create --username admin --firstname Ivan --lastname Kozlov --role Admin --email kozlov.titan2010@yandex.ru
```

##________ЗАПУСК (Сервер). Терминал1. Ура!

```
airflow webserver -p 8080
```


##________ЗАПУСК (Шедулер). Терминал2. Ура!
```
pipenv shell 
airflow scheduler
```

