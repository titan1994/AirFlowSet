#Настройка AirFlow Linux+Postgres

##________Устновка
После установки по ссылке:

https://airflow.apache.org/docs/apache-airflow/stable/installation.html
А там нихрена не понятно... 
Просто вобщем напишите в любой папке, куда упадут пипы: 

```
pipenv install apache-airflow
pipenv shell
```

Потом добавьте домашнюю директорию в path:

```
export AIRFLOW_HOME=~/airflow
```

Потом сразу инит базы, чтобы возникли все нужные файлы 

```
airflow db init
```

##________Настройка каталогов
https://www.astronomer.io/guides/airflow-importing-custom-hooks-operators

Добавление папок dags, plugins, hooks, operators, sensors. И тестовых файлов. Чтобы понять как 
создать собственный хук, сенсор, оператор и прикрутить к дагу. 
Единственный способ!

```
├── airflow.cfg
├── airflow.db
├── dags
│   └── my_dag.py
└── plugins
    ├── hooks
    │   └── my_hook.py
    ├── operators
    │   └── my_operator.py
    └── sensors
        └── my_sensor.py
```

my_hook.py:

```
from airflow.hooks.base_hook import BaseHook

class MyHook(BaseHook):

    def my_method(self):
        print("Hello World")
```

my_sensor.py:

```
from airflow.sensors.base_sensor_operator import BaseSensorOperator
from airflow.utils.decorators import apply_defaults


class MySensor(BaseSensorOperator):

    @apply_defaults
    def __init__(self,
                 *args,
                 **kwargs):
        super(MySensor, self).__init__(*args, **kwargs)

    def poke(self, context):
        return True
```

my_operator.py:

```
from airflow.operators.base_operator import BaseOperator
from airflow.utils.decorators import apply_defaults
from hooks.my_hook import MyHook


class MyOperator(BaseOperator):

    @apply_defaults
    def __init__(
            self,
            name: str,
            **kwargs) -> None:
        super().__init__(**kwargs)
        self.name = name

    def execute(self, context):
        message = f"Hello {self.name}"
        print(message)
        hook = MyHook('my_conn')
        hook.my_method()
        return message
```

my_dag.py:

```
from airflow import DAG
from datetime import datetime, timedelta
from operators.my_operator import MyOperator
from sensors.my_sensor import MySensor

default_args = {
    'owner': 'airflow',
    'depends_on_past': False,
    'start_date': datetime(2018, 1, 1),
    'email_on_failure': False,
    'email_on_retry': False,
    'retries': 1,
    'retry_delay': timedelta(minutes=5),
}

with DAG('example_dag',
         max_active_runs=3,
         schedule_interval='@once',
         default_args=default_args) as dag:
    sens = MySensor(
        task_id='taskA'
    )

    op = MyOperator(
        name='Some name',
        task_id='taskB'
    )

    sens >> op
```
       
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
drop database airflow;
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

##________Правим airflow.cfg - Ключ шифрования (Если его нет). fernet_key. (КЛЮЧ ЕСТЬ ВСЕГДА. НИЧЕГО НЕ ДЕЛАТЬ)
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

##________Проект в пайчарм (ПРОЧИТАТЬ И НЕ ДЕЛАТЬ ТАК - ВСЁ СЛОМАЛОСЬ!!!)
Чтобы голова не сошла с ума от изобилия - заводим новый проект в пайчарм и указываем 
папку airflow ему как папку проекта. Чтобы можно было в гит пулять даги, плагины и прочее. 

Если вдруг папка dags - не пуляется в гит - создайте другую папку и перепешите конфиг airflow.cfg:

```
dags_folder = /home/uadmin/airflow/dags_ex
```

у меня не пулялась в гит. 100500 раз пробовал пересоздавать проекты... 
возможно в этом и проблема, что пайчарм уже сошёл с ума. 
но вылечил я это всё создав папку dags_ex

##________Примеры плагинов, дагов и операторов (ИЗ ПОЛЕЗНОГО - НУЖНО РЕБУТАТЬСЯ ПОСЛЕ ИЗМЕНЕНИЙ ПЛАГИНОВ)

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



##________Самый правильный пример структуры проекта (НАДО ДЕЛАТЬ ВНАЧАЛЕ!!!)

https://www.astronomer.io/guides/airflow-importing-custom-hooks-operators

Жаль мы к нему пришли спустя время. Засела дрянь:
```
Broken DAG: [/home/uadmin/airflow/dags_ex/test_operators.py] Traceback (most recent call last):
  File "<frozen importlib._bootstrap>", line 219, in _call_with_frames_removed
  File "/home/uadmin/airflow/dags_ex/test_operators.py", line 3, in <module>
    from airflow.operators.my_operators.py import HelloOperator
ModuleNotFoundError: No module named 'airflow.operators.my_operators'
```

Не помог рестарт убунты, сервера, переинициализация. Ничего не помогает.
Папку изменил уже давно в конфиге, но всё равно за дагами он лезет вот сюда:  
/home/uadmin/airflow/dags_ex

Будем переутсанавливать. 
Остался последний варик - снести папку airflow и настроить всё заново. 
Так же для верности - удалим и создадим заново таблицу в постгресе