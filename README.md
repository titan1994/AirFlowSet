#Настройка AirFlow Linux+Postgres
После установки по ссылке:

https://airflow.apache.org/docs/apache-airflow/stable/installation.html

Нас ждут весёлые старты, описанные далее. Без этого не взлетит

##________Нельзя без этих инструментов. Редакторы кода:
```
sudo snap install pycharm --classic
sudo snap install sublime-text --classic

```
Вырубить блокировщик экрана
```

gsettings set org.gnome.desktop.screensaver lock-enabled false
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

Плагин чтобы победить Warning при проектировании дагов:
```
pip install apache-airflow['cncf.kubernetes']
```

##________Правим airflow.cfg - База данных для postgres 

экзекьютор меняем на такой:
```
executor = LocalExecutor
```
а также коннектор к базе прописываем на постгрес:
```
sql_alchemy_conn = postgresql+psycopg2://admin:admin@127.0.0.1:5432/airflow
```

##________Правим airflow.cfg - Ключ шифрования (Если его нет). fernet_key
Предварительно проверить нет ли там ключа. Если есть - можно пропустить этот танец
Открываем python в консоли (помним что мы внутри окружения pipenv shell):

```
python
from cryptography.fernet import Fernet
key = Fernet.generate_key()
print(key)
exit(0)
```

Копируем ключ и вставляем в airflow.cfg 

```
fernet_key=key
```

##________Правим webserver_config.py(раскомментируем как тут 3 строки)
```
# Uncomment to setup Full admin role name
AUTH_ROLE_ADMIN = 'Admin'

# Uncomment to setup Public role name, no authentication needed
AUTH_ROLE_PUBLIC = 'Public'

# The default user self registration role
AUTH_USER_REGISTRATION_ROLE = "Public"
```

##________Создаем папку airflow/dags
Просто создаем папку, потому что:  
В файле настроек airflow.cfg есть параметр dags_folder, 
он указывает на путь, где лежат файлы с DAGами. Это путь 
$AIRFLOW_HOME/dags. Именно туда мы положим наш код с задачами.

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

##________Продакшн запуск. Убираем примеры airflow.cfg
Когда наигрались с примерами (они в большинстве своём просто поглазеть)
load_examples в  airflow.cfg  поставить в значение False
```
load_examples = False
```

##________Проект в пайчарм
Чтобы голова не сошла с ума от изобилия - заводим новый проект в пайчарм и указываем 
папку airflow ему как папку проекта. Чтобы можно было в гит пулять даги, плагины и прочее. 

Если вдруг папка dags - не пуляется в гит - создайте другую папку и перепешите конфиг airflow.cfg:

```
dags_folder = /home/uadmin/airflow/dags_ex
```

у меня не пулялась в гит. 100500 раз пробовал пересоздавать проекты... 
возможно в этом и проблема, что пайчарм уже сошёл с ума. 
но вылечил я это всё создав папку dags_ex

##________Примеры плагинов, дагов и операторов

НЕ РАБОТАЕТ СОЗДАНИ ОПЕРАТОРОВ. УСТАРЕЛ МАНУАЛ. 
Нужен свой оператор? Идём сюда:
https://airflow.apache.org/docs/apache-airflow/stable/howto/custom-operator.html

Вот тут всё пошагово верно но синтаксис они поменяли:
https://michal.karzynski.pl/blog/2017/03/19/developing-workflows-with-apache-airflow/

Чтобы протестировать своего нового оператора, 
вы должны остановить (CTRL-C) и перезапустить веб-сервер 
и планировщик Airflow. После этого вернитесь в пользовательский 
интерфейс Airflow, включите my_test_dag DAG и запустите запуск. 
Взгляните на журналы для my_first_operator_task.

Даже можно прикрутить отлдчик пайчарма через IPython. Вобщем - читаем ссылку

##________Самый правильный пример структуры проекта

https://www.astronomer.io/guides/airflow-importing-custom-hooks-operators