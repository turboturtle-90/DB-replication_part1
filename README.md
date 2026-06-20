# Домашнее задание к занятию «Репликация и масштабирование. Часть 1» - `Смирнов Максим`

### Задание 1

На лекции рассматривались режимы репликации master-slave, master-master, опишите их различия.




### Задание 2

`Проверка записи через мастер - добавлена строка 'Привет! Репликация работает!'. Также сделан запрос на статус - по выводу видно, что пользователь replicator иммет доступ на локал хосте, т.е. репликация настроена`

![master-insert1.jpg](https://github.com/turboturtle-90/DB-replication_part1/blob/b8bb7f154bdbb1f690b2e30c59d30ad55b6e2d2d/master-insert1.jpg)


`Проверка, что в реплику пишется информация с мастера, но не может писаться напрямую `

![slave-read.jpg](https://github.com/turboturtle-90/DB-replication_part1/blob/b8bb7f154bdbb1f690b2e30c59d30ad55b6e2d2d/slave-read.jpg)

`Текст команд для конфигурирования master-slave системы на postgres приведен ниже :`

sql команды (выполняеются после вызова postgres командой строки sudo -i -u postgres psql )

```                                                         
CREATE DATABASE test_master;
CREATE USER replicator WITH REPLICATION ENCRYPTED PASSWORD '123456';
SHOW data_directory;
```

Запрос пути где живет файл конфигурации
```                                                         
sudo -i -u postgres psql -c "SHOW data_directory;"
```

В файл кофигурации дописать если еще не дописаны права на подключение пользователя replication локально
```                                                         
sudo nano /etc/postgresql/16/main/pg_hba.conf

host    replication     replicator      127.0.0.1/32            scram-sha-256
```

В настройках master активируем репликацию

```                                                         
sudo nano /etc/postgresql/16/main/postgresql.conf

wal_level = replica
max_wal_senders = 10
hot_standby = on
```


Перезагрузка сервера
```                                                         
sudo systemctl restart postgresql
```

Создаем новую пустую папку, где будут жить данные нашей реплики (создавать в директории полученной sql-командой SHOW data_directory)
```                                                         
sudo mkdir -p /var/lib/postgresql/16/replica
sudo chown postgres:postgres /var/lib/postgresql/16/replica
sudo chmod 700 /var/lib/postgresql/16/replica
```

Копируем файл конфигурации из master папки в slave папку
``` 
sudo cp /etc/postgresql/16/main/postgresql.conf /var/lib/postgresql/16/replica/postgresql.conf
sudo chown postgres:postgres /var/lib/postgresql/16/replica/postgresql.conf
```

Редактируем файл - меняем порт, т.к. мастер и слейв слушают разные порты, также меняем настройки чтобы избежать конфликтов
```
sudo nano /var/lib/postgresql/16/replica/postgresql.conf

port = 5433
data_directory = '/var/lib/postgresql/16/replica'
sudo nano /var/lib/postgresql/16/replica/postgresql.conf
ssl = off
#external_pid_file = '/var/run/postgresql/16-main.pid'
```
Добавляем папку conf.d
sudo mkdir -p /var/lib/postgresql/16/replica/conf.d
sudo chown postgres:postgres /var/lib/postgresql/16/replica/conf.d

Запускаем. Если что то не так читаем логи 
```
sudo -i -u postgres /usr/lib/postgresql/16/bin/pg_ctl -D /var/lib/postgresql/16/replica -l /var/lib/postgresql/16/replica/logfile start
sudo tail -n 20 /var/lib/postgresql/16/replica/logfile
```

Когда slave запущен, проверяем работоспособность - запишем что-то в базу через мастер и проверим, что в реплику записалось то же самое. 
Запись через мастер
```
sudo -i -u postgres psql -p 5432 -d test_master

CREATE TABLE replication_test (id serial PRIMARY KEY, message text);
INSERT INTO replication_test (message) VALUES ('Привет! Репликация работает!');
```

Проверка на slave
```
sudo -i -u postgres psql -p 5433 -d test_master

SELECT * FROM replication_test;
```
Как результат должны увидеть запись 'Привет! Репликация работает!'

---
